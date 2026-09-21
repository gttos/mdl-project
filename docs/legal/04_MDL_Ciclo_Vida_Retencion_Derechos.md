# 4. Ciclo de vida, conservación y ejercicio de derechos

> Modelo funcional objetivo de la beta a 21 de septiembre de 2026. Los plazos jurídicos, excepciones y lenguaje público deben ser aprobados por la asesoría y contrastados con la configuración real.

## 1. Principios operativos

- conservar sólo datos necesarios para una finalidad aprobada;
- distinguir uso activo, borrado funcional, purga, backup y conservación jurídica;
- no presentar soft delete como destrucción física inmediata;
- no presentar seudonimización como anonimización;
- coordinar estructura, Vault, dispositivo y proveedores;
- no copiar contenido íntimo a tickets, emails, logs o expedientes de derechos;
- demostrar qué se eliminó, qué se retiene, por qué y hasta cuándo.

## 2. Ciclo de vida

1. **Alta:** email crea una cuenta provisional; Apple, sólo en iOS, puede crear una cuenta verificada sin contraseña MDL.
2. **Verificación:** la provisional sólo accede al flujo de email durante siete días; los reenvíos no amplían el plazo.
3. **Uso:** API conserva cuenta y estructura; el cliente cifra/descifra el Vault localmente.
4. **Rectificación:** una edición sustituye estructura y/o una revisión cifrada conforme a la frontera durable.
5. **Archivo:** oculta un Crush de la lista activa sin eliminar su historia.
6. **Borrado funcional:** retira la entidad del uso ordinario después de confirmación.
7. **Purga:** destruye filas u objetos elegibles tras su ventana técnica.
8. **Baja:** revoca acceso, destruye contenido privado y aplica a estructura/evidencias la política aprobada.
9. **Backups:** expiran por un ciclo propio y no reintroducen datos al uso ordinario tras una restauración.

## 3. Matriz de conservación para aprobación

| Categoría | Regla funcional o propuesta | Acción al borrar/baja | Decisión o evidencia necesaria |
|---|---|---|---|
| Cuenta activa | mientras exista cuenta y finalidad | retirar identidad y desactivar | base, campos necesarios y destino posterior |
| Cuenta provisional | siete días fijos desde alta | baja íntegra y liberación del email | scheduler y prueba de borrado |
| Email de cuenta | cifrado recuperable durante vida de cuenta | retirar email e índice | keyring, rotación, backups y evidencia |
| Sesión | siete días desde emisión | revocación inmediata en logout/baja | pruning efectivo |
| Identidad Apple | mientras exista vínculo | retirar y solicitar revocación al dar de baja | transparencia, Private Relay y evidencia; sólo iOS en beta |
| Continuación Apple | diez minutos o primer uso | purga tras uso o vencimiento | token público no persistido |
| Revocación Apple fallida | máximo 30 días tras fallo final | reintento manual autorizado y eliminación al vencer | alerta, tarea y prueba |
| OTP | 15 minutos; cinco intentos; bloqueo 15 minutos | invalidación tras uso, intentos o expiración | pruning y anti-enumeración |
| Inactividad | 18 meses desde `last_login` o `email_verified_at`, más 20 días | baja con el mismo resultado que la voluntaria | aprobación de base, aviso y falta de entrega |
| Aceptaciones | mientras sean necesarias para demostrar versión | posible conservación probatoria | plazo y contexto mínimo |
| Estructura activa del diario | mientras exista entidad/cuenta | borrar o seudonimizar según flujo | necesidad por campo |
| Snapshot Vault vigente | mientras exista entidad privada | retirada coordinada | acceso, portabilidad y backups |
| Revisión sustituida | hasta confirmar sustituta | retiro después de confirmación | monitorización de huérfanos |
| Subida Vault incompleta | 24 horas desde creación | liberar cuota al vencer y limpiar objeto/metadata | ejecución y evidencia operativa |
| Blob retirado | siete días | destrucción programada | reintentos, versiones y backups |
| Binding opaco | hasta destruir la última revisión asociada | eliminar al completar purga | idempotencia y evidencia |
| Fotos locales | mientras exista foto/Crush/cuenta | borrar en el dispositivo | exclusión de backup y file protection |
| Activity/Audit/Auth/Security | plazo por aprobar | purga programada o hold autorizado | DEC-09 |
| Logs y proveedores | configuración real | expirar conforme al proveedor | retención, región, DPA y scrubbing |
| Backups RDS/S3 | configuración real | expiración y bloqueo de reactivación | inventario, plazo y restore |
| Feedback y recibo de baja | sólo finalidad aprobada | conservación limitada | campos, acceso y plazo |
| Estructura post-baja | sólo si Legal autoriza finalidad estadística | separación del uso ordinario | DEC-02 completa |

Un plazo implementado no es automáticamente un plazo jurídico correcto. Si Legal lo modifica, deben cambiar configuración, tareas, pruebas y textos públicos.

## 4. Baja por inactividad

Producto define este comportamiento:

1. Los 18 meses se cuentan desde `last_login`; si no existe en una cuenta verificada, desde `email_verified_at`.
2. Si faltan ambas fechas, no se cierra automáticamente y se genera una anomalía técnica investigable.
3. El plazo de 20 días comienza cuando el proveedor acepta el aviso para entrega; no se registran aperturas ni clics.
4. Los fallos temporales se reintentan de forma finita.
5. Un fallo general del proveedor o de configuración no inicia la gracia.
6. Un rechazo definitivo de esa dirección, incluido hard bounce o Private Relay desactivado, inicia los 20 días desde el último intento tras agotar los reintentos.
7. Sólo un login correcto durante la gracia cancela el cierre.

Legal debe aprobar base, transparencia, suficiencia del aviso y tratamiento del caso no entregado antes de activar el cierre automático.

## 5. Borrado por alcance

### 5.1 Foto

La aplicación elimina del dispositivo el archivo saneado, la cache o thumbnail y sus referencias locales.

### 5.2 Experiencia e hito

- Una experiencia independiente puede eliminarse sin límite de antigüedad y sin borrar su período.
- Una experiencia vinculada no se elimina sola: inicia el borrado conjunto con su hito y queda sujeta a `latest-only`.
- Desde un último hito posterior elegible se puede eliminar sólo el cambio, conservando su experiencia como independiente, o eliminar ambas partes.
- La experiencia inicial sólo desaparece con el Crush.
- Las revisiones Vault afectadas se retiran y se purgan después de siete días.

### 5.3 Período

- El período inicial sólo desaparece con el Crush.
- Sólo puede eliminarse el período posterior más reciente cuando su hito inicial es su único hito.
- Puede contener experiencias independientes posteriores: todas se eliminan con período, hito inicial y experiencia inicial del período.
- El período anterior permanece cerrado y nunca se reabre.

La restricción funcional `latest-only` no limita los derechos jurídicos; una solicitud de derechos utiliza un procedimiento distinto de los botones ordinarios.

### 5.4 Crush

El borrado abarca perfil, períodos, hitos, experiencias, actividades relacionadas, bindings, revisiones remotas, entradas/intenciones locales y `LocalMedia/{crushId}` o equivalente. Archivar no ejecuta estas acciones.

### 5.5 Cuenta

La baja es irreversible y debe confirmarse como operación coherente. Retira email e índice, password, fecha de nacimiento, género/país, Apple, sesiones, OTP, dispositivos, encapsulaciones y recovery. El Vault y sus blobs salen del uso ordinario de inmediato y se purgan después de siete días; los bindings se conservan sólo hasta completar la purga.

El cliente elimina almacén seguro, SQLite, pendientes, preferencias sensibles y fotos únicamente después de confirmar el resultado remoto. Perder la respuesta se resuelve mediante el secreto de confirmación de baja sin exponer email o ID de cuenta.

No existe reset independiente del Vault. Restaurar un backup no puede reactivar una cuenta ni su diario.

## 6. Estructura después de la baja

Producto pretende conservar fuera del uso ordinario estructura seudonimizada de Crushes, experiencias y relaciones para posibles agregados. No se llama anónima y no se utiliza hasta obtener decisión jurídica.

Legal debe resolver:

- finalidad y base;
- campos estrictamente necesarios;
- plazo máximo;
- acceso y separación física/lógica;
- supresión de UUIDs, fechas exactas y combinaciones raras;
- umbrales de agregación y pruebas de reidentificación;
- atención de derechos y posibles holds;
- eliminación total si no existe justificación suficiente.

## 7. Derechos de la persona usuaria

El procedimiento deberá autenticar proporcionalmente, registrar alcance sin copiar contenido Vault, localizar cuenta/estructura/metadata/proveedores, distinguir lo legible por backend de lo descifrable sólo en el dispositivo y atender acceso, rectificación, restricción, oposición, supresión y portabilidad conforme al criterio aprobado.

Backend puede exportar cuenta y estructura, pero no plaintext Vault. Una exportación legible completa requeriría generación local validada, formato interoperable, protección del archivo y una advertencia de que fuera de la app ya no queda protegido por el Vault. No debe prometerse hasta estar implementada y validada.

## 8. Solicitudes de terceras personas

Legal debe definir cómo acreditar identidad y vínculo sin pedir datos excesivos, cuándo MDL debe o puede buscar, cómo equilibrar derechos del tercero con la privacidad de la cuenta, cómo evitar confirmar la existencia del diario y qué hacer cuando no sea posible identificar razonablemente el registro.

No se accede indiscriminadamente al Vault ni se solicita a la persona usuaria su secreto de recovery.

## 9. Holds, incidentes y proveedores

Cualquier excepción de conservación debe identificar autorización, finalidad, datos, responsable, revisión y vencimiento. La asesoría debe definir supuestos de litigio, fraude, seguridad u obligación legal y el procedimiento de incidentes aplicable.

## 10. Evidencia previa al lanzamiento

- pruebas de borrado de foto, experiencia, hito, período, Crush y cuenta;
- prueba de purga a siete días y de subidas incompletas a 24 horas;
- inventario de backups/versiones y restore sin reactivación;
- retención efectiva de eventos y observabilidad;
- configuración real de email, rebotes e inactividad;
- decisión firmada sobre estructura post-baja;
- procedimiento probado para persona usuaria y tercera persona;
- exportación local validada o texto público que describa honestamente su ausencia;
- aprobación de plazos, excepciones y lenguaje público.
