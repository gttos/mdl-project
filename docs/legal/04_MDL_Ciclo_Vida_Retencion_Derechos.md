# 4. Ciclo de vida, conservación y ejercicio de derechos

> **Estado:** actual, verificado el 5 de agosto de 2026.
> **Importante:** los plazos técnicos existentes no constituyen por sí solos una política aprobada, y algunas conductas descritas acá son defectos conocidos, no diseño deseado.

## 1. Conceptos utilizados

| Concepto | Significado en MDL |
|---|---|
| Activo | Dato visible y utilizado para prestar el servicio |
| Revocado | Artefacto conservado temporalmente pero ya no autorizado para uso, como una sesión o dispositivo |
| Soft delete | Registro oculto para el producto pero todavía presente en la base |
| Purga | Eliminación física de la fila u objeto operativo |
| Borrado local | Eliminación en un dispositivo concreto |
| Seudonimización | Reducción de identificadores directos con posibilidad de reidentificación |
| Anonimización | Sobrescritura irreversible de campos con placeholders (ej. `deleted_{id}@anon.anon`); los registros anonimizados se conservan para estadísticas agregadas |

## 2. Ciclo de vida general hoy

1. **Recogida:** el usuario introduce datos en un único formulario por entidad; los campos funcionales y el contenido de contacto/lugar/descripción/nota viajan juntos por HTTP.
2. **Persistencia dual:** la API guarda los campos funcionales en texto plano, cifra con la clave de aplicación los campos de contacto/lugar/descripción/nota, y —si el Vault está activo— iOS cifra en paralelo una copia con VMK.
3. **Actualización:** el valor se corrige; en Vault la versión anterior permanece hasta confirmar la nueva, salvo el bug de recovery descrito en el documento 03.
4. **Solicitud de borrado:** se revocan accesos y se marcan las entidades remotas afectadas.
5. **Purga operativa:** se eliminan filas y objetos del Vault tras la ventana técnica (7 días para blobs).
6. **Expiración de copias:** backups, réplicas y logs del proveedor según su configuración, no auditada en este expediente.

## 3. Matriz de conservación

| Categoría | Regla técnica hoy | Inicio del plazo | Efecto esperado del borrado | Punto que debe validar la asesoría |
|---|---|---|---|---|
| Cuenta activa | Mientras se presta el servicio | Alta | Anonimización + soft delete | Plazo tras baja y excepciones justificadas |
| Tokens y sesiones | Hasta expiración (30 días) o revocación | Emisión | `sanctum:prune-expired` diario | Plazo máximo y logs asociados |
| Códigos de recuperación (OTP) | Purgables 24 horas después de expirar | Expiración | Eliminar registro | Suficiencia y plazo del proveedor de correo |
| Crush, experiencia y relación | Soft delete; purga técnica documentada a 7 días (a confirmar contra el comando real) | Solicitud de borrado | Eliminar estructura funcional, **incluyendo las columnas cifradas con clave de aplicación** | Si siete días es proporcionado |
| Blob Vault borrado | Purga técnica tras 7 días (`vault:purge-blobs`, diario) | Soft delete | Eliminar objeto S3 y fila de metadata | Backups y versiones del proveedor |
| Dispositivo y recovery key | Mientras estén autorizados o sean recuperables | Autorización/creación | Revocar o eliminar al sustituir/baja | Historial mínimo de seguridad |
| **Fotografías** | **Hasta borrado individual manual únicamente** | Creación local | **No se elimina automáticamente al borrar el crush ni la cuenta** | Debe corregirse antes de anunciar borrado completo al usuario |
| Eventos de actividad | Documentado como purga >7 días salvo cinco últimos por usuario | Creación | Cascade con entidad asociada | Confirmar comando real |
| Auditoría (`AuditEvent`) | Documentado a 90 días | Creación | Purga automática | Confirmar comando real y proporcionalidad |
| Alertas de seguridad (`SecurityAlert`) | Documentado a 30 días | Creación | Purga automática | Confirmar comando real |
| Aceptaciones legales | Sin poda automática identificada | Aceptación | Se conserva aunque la cuenta esté en soft delete | Plazo probatorio |
| Feedback de baja | Sin poda automática identificada | Envío | — | Texto libre, soporte y plazo |
| Backups y réplicas de infraestructura | No definido por el repositorio de aplicación | Copia | Depende de configuración de AWS RDS/S3, no auditada acá | Plazo máximo por sistema y proveedor |

La existencia de una regla en el código no demuestra que la purga se ejecute correctamente en producción; debe verificarse con evidencia operativa antes de afirmarlo públicamente.

## 4. Borrado de una entidad

### 4.1 Experiencia o relación

El borrado trata como una unidad: fila funcional en API-DB (incluyendo columnas cifradas con clave de aplicación), entrada Vault local si existe, versión o versiones remotas del Vault, y eventos asociados. Un error remoto en el Vault conserva las referencias necesarias y deja la operación pendiente de reintento.

### 4.2 Crush

Al borrar un crush hoy:

- se aplica soft delete a la fila funcional del crush, sus experiencias y relaciones, incluyendo las columnas cifradas con clave de aplicación;
- iOS intenta borrar cada blob remoto del Vault y continúa si falla;
- se eliminan entradas Vault locales y notas locales del dispositivo;
- **la carpeta local de fotografías `Documents/LocalMedia/<crushId>/` no se elimina** y permanece en el dispositivo hasta que el usuario la borre manualmente o desinstale la app.

La interfaz no debe presentarse como si el borrado incluyera las fotografías mientras este comportamiento no cambie.

## 5. Baja integral de cuenta

Según las reglas de negocio verificadas en el código (BR-60 a BR-69):

1. la anonimización es irreversible: los valores originales no pueden recuperarse;
2. el email se reemplaza por `deleted_{id}@anon.anon`;
3. el nombre del crush se reemplaza por `ANON_{random}`;
4. los campos PII se sobrescriben con placeholders;
5. los registros anonimizados se preservan para estadísticas agregadas;
6. el soft delete del usuario se ejecuta después de la anonimización;
7. los tokens Sanctum se revocan de inmediato;
8. la eliminación de archivos S3 legacy es asíncrona vía job con reintentos (máximo 3);
9. los `ActivityEvent` del usuario se eliminan en cascada;
10. la anonimización de cada crush está protegida por try/catch: el fallo de uno no interrumpe los demás.

**No verificado en esta revisión:** si el Vault (blobs, devices, recovery keys) y las fotografías locales quedan cubiertos por el mismo flujo de baja integral, o si requieren pasos separados. Debe confirmarse con ingeniería antes de describir la baja como completa en la política pública. El `google_id` puede conservarse tras la anonimización según una decisión de negocio no cerrada (ver sección 20 de `api/BBP-MyDuckingLife.md`).

Un dispositivo offline o perdido no está bajo control remoto absoluto de MDL.

## 6. Backups, réplicas y almacenamiento remoto

No auditado en este expediente: configuración real de backups de RDS, versionado y lifecycle de S3, retención de logs de Sentry y del proveedor de correo. Debe solicitarse a operaciones antes de firmar cualquier plazo público.

## 7. Procedimiento para derechos del usuario

No existe hoy un procedimiento operativo documentado y ejecutado para derechos del usuario más allá de: edición directa del perfil propio, edición o borrado de crushes/experiencias/relaciones, y baja de cuenta con anonimización. **No existe un endpoint ni flujo de exportación de datos personales (portabilidad).** El siguiente procedimiento es una propuesta a validar, no una descripción de lo que ocurre hoy:

1. **Recepción:** registrar solicitud, canal, fecha y alcance.
2. **Identidad:** usar sesión autenticada cuando sea posible.
3. **Búsqueda:** localizar API-DB (incluyendo columnas cifradas con clave de aplicación), blobs del Vault, proveedores, logs y dispositivo accesible.
4. **Clasificación:** separar datos del usuario y datos sobre terceras personas.
5. **Ejecución:** acceder, rectificar, exportar o suprimir según la respuesta jurídica.
6. **Respuesta:** lenguaje claro, límites explicados.

## 8. Acceso y portabilidad del Vault

El backend puede exportar de forma legible la cuenta, la estructura funcional (incluyendo el contenido de las columnas cifradas con clave de aplicación, que sí puede descifrar) y aceptaciones. No puede convertir el ciphertext del Vault en texto sin la VMK del usuario. **No existe hoy ninguna herramienta, endpoint ni flujo en iOS que genere una exportación legible para el usuario.** Cualquier afirmación pública de portabilidad debe esperar a que este flujo exista.

## 9. Rectificación

| Dato | Mecanismo hoy |
|---|---|
| Cuenta | Edición directa |
| Estructura funcional y contenido de contacto/lugar/descripción/nota | Edición de la entidad; se sobrescribe la columna funcional y, si aplica, se crea una nueva versión en el Vault |
| Fotografía | Sustitución o borrado local |
| Log histórico | No se reescribe de forma general |

## 10. Solicitudes de una tercera persona

La API no dispone de una búsqueda global fiable por nombre en el Vault, pero el nombre del crush **sí es buscable directamente en la base de datos funcional**, porque se guarda en texto plano. Esto cambia la evaluación de riesgo respecto de un modelo puramente E2EE: una solicitud de una tercera persona, o un acceso indebido a la base de datos, puede localizar su nombre sin necesitar ninguna clave.

## 11. Restricción, oposición y decisiones automatizadas

No implementado hoy más allá del borrado de la entidad completa. La asesoría debe identificar qué tratamientos admiten oposición y si alguna presentación de métricas requiere revisión humana.

## 12. Evidencia operativa pendiente

- confirmación de ejecución real, fallos y reintentos de las tareas de poda (`model:prune`, `sanctum:prune-expired`, `accounts:process-inactive`, `vault:purge-blobs`);
- antigüedad por tabla y número de registros soft-deleted en producción;
- prueba de que el borrado de un crush efectivamente elimina las columnas cifradas con clave de aplicación y no solo las oculta;
- corrección y prueba del borrado de la carpeta de fotografías al borrar un crush;
- decisión y construcción de un flujo real de exportación de datos;
- confirmación del tratamiento del Vault y las fotos dentro de la baja integral de cuenta.

## 13. Referencia normativa para la revisión

La asesoría debe contrastar este ciclo de vida con el [texto oficial del RGPD en EUR-Lex](https://eur-lex.europa.eu/eli/reg/2016/679/oj?locale=es), en particular los principios, derechos, seguridad, brechas y evaluación de impacto. Este documento no determina por sí mismo qué plazo o excepción resulta jurídicamente válido.
