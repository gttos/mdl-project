# 3. Vault, almacenamiento y seguridad

> **Estado:** actual, verificado el 5 de agosto de 2026. El Vault está activo pero es parcial y coexiste con columnas legacy legibles por el backend.
> **Objetivo:** explicar qué puede leer cada sistema hoy y qué controles existen realmente, sin exigir conocimientos de criptografía o desarrollo.

## 1. Qué es el Vault hoy

El Vault es un sistema en construcción que cifra parte del contenido textual privado de MDL. Cuando está activo para una cuenta y una entrada ya fue migrada:

- iOS crea y custodia la clave que permite abrirla (VMK);
- iOS cifra esa entrada antes de enviarla;
- la API autoriza al propietario y conserva metadata operativa;
- el almacenamiento remoto conserva el paquete cifrado;
- la lectura y composición de la pantalla ocurren en el dispositivo después del desbloqueo.

La API no ofrece una operación para descifrar blobs y no recibe la VMK en claro. **Esto es cierto únicamente para el contenido que pasó por el flujo del Vault.** El mismo contenido también existe, para toda cuenta que lo haya introducido por el formulario normal, en columnas funcionales que la API sí puede descifrar con una clave propia de aplicación — ver el [inventario maestro](02_MDL_Inventario_Maestro_Datos.md).

## 2. Qué entra en el Vault hoy

| Tipo de entrada | Contenido principal | Blob type usado hoy |
|---|---|---|
| Crush | Nombre, profesión, contacto, redes y fecha de nacimiento | `text` |
| Experiencia | Lugar y descripción | `text` |
| Relación | Nota de cierre | `text` |

El schema de la API admite blobs `text`, `image` y `video`, pero la cola activa de iOS solo crea blobs `text`. No hay fotos ni ningún otro archivo en el Vault hoy.

Las fechas, estados, clasificaciones y valoraciones necesarias para operar el producto permanecen en la base funcional y son legibles por la API, cifradas o no según el campo.

## 3. Flujo de escritura hoy

1. El usuario introduce o modifica contenido en un único formulario de iOS.
2. Ese formulario envía a la API, en el mismo request, tanto los campos funcionales como el contenido de contacto/lugar/descripción/nota.
3. La API persiste ese contenido en columnas funcionales, cifrándolo con la clave de aplicación cuando corresponde.
4. Si el usuario tiene el Vault activo, la app además cifra una copia del mismo contenido con la VMK.
5. La app registra en la API metadata técnica y un ID remoto aleatorio para esa copia.
6. La API comprueba que la cuenta es propietaria del Vault y entrega una URL temporal (15 minutos).
7. iOS sube el ciphertext directamente al almacenamiento de objetos.

**Los endpoints funcionales sí transportan hoy los campos privados**: nombre, contacto, redes, lugar, descripción y nota de cierre viajan por HTTP en el mismo request que los datos funcionales, con o sin Vault activo.

## 4. Flujo de lectura y recuperación

1. iOS obtiene desde la API la estructura funcional, que ya incluye el contenido de las columnas cifradas con clave de aplicación descifrado.
2. Si la entrada también existe en el Vault, iOS puede descargar el ciphertext del almacenamiento remoto y descifrarlo localmente con la VMK.
3. La recuperación del Vault agrupa las entradas por `crushId`: cuando un mismo crush tiene varias experiencias o relaciones, la lógica actual conserva solo la de mayor secuencia y **solicita el borrado remoto de las demás**. Esto puede eliminar entradas válidas del Vault durante una recuperación.

## 5. Claves y recuperación

### 5.1 Clave principal (VMK)

- La VMK es aleatoria y tiene 32 bytes.
- Se guarda en Keychain con `WhenUnlockedThisDeviceOnly` y biometría en dispositivo físico.
- En memoria solo permanece durante las operaciones que la necesitan.

### 5.2 Encapsulación

La VMK se protege con material derivado de un secreto de recuperación mediante Argon2id (65.536 KiB de memoria, 3 iteraciones, paralelismo 4). La API conserva la VMK encapsulada, nunca la VMK en claro.

### 5.3 Métodos de recuperación

| Método | Estado hoy |
|---|---|
| Contraseña de recuperación | Funcional: la app descarga la encapsulación por contraseña desde `GET /api/vault` y reconstruye la VMK localmente. |
| Frase de 24 palabras | La encapsulación mnemónica que la app genera en el alta **históricamente no se persistía** al llegar a `POST /api/vault/init`. El código ya incorpora una tabla y endpoints dedicados (`GET`/`PUT /vault/encapsulation/mnemonic`) y una lectura real desde iOS, pero **todavía no se validó de punta a punta con build y prueba manual en dispositivo**. Hasta esa validación, no debe asumirse que este método reconstruye el mismo Vault que la contraseña en un dispositivo nuevo. |
| Transferencia QR | ECDH P-256 entre dispositivos y descarga posterior de blobs. |

El material mostrado al usuario debe tratarse como un secreto. MDL debe explicar que una persona con acceso a ese secreto y a la cuenta podría recuperar el contenido del Vault, y que la pérdida de todos los mecanismos puede volverlo irrecuperable — sin afectar el contenido que además existe en las columnas funcionales, que sigue siendo accesible por el backend independientemente del estado del Vault.

## 6. Metadata que sigue siendo visible

Aunque el contenido del Vault esté cifrado, API, base de datos y almacenamiento remoto pueden conocer: ID de cuenta, Vault y blob; tamaño, hash y versión criptográfica; secuencia y fechas; ruta y proveedor de almacenamiento; cantidad de blobs y frecuencia de uso; dispositivos autorizados; existencia de material de recuperación; IP, user agent, rutas y eventos de seguridad asociados.

## 7. Almacenamiento por capa

| Capa | Datos | Control principal | Riesgo residual hoy |
|---|---|---|---|
| Keychain iOS | VMK y clave privada del dispositivo | Controles de acceso del sistema, desbloqueo y biometría | Dispositivo comprometido o sesión desbloqueada |
| SQLite iOS | Ciphertext del Vault, nonce, tag, cola, **y el índice `vault_search_index` en texto plano** | Sandbox y protección de archivos | El índice de búsqueda es texto plano por diseño actual, no por bug de logging: cualquiera con acceso forense al archivo lee nombre, profesión, lugar, descripción y nota de cierre sin necesitar la VMK |
| Memoria iOS | Contenido descifrado, búsqueda | Sesión Vault | Puede aparecer en logs locales (`LegacyMigrationOrchestrator`, `CrushViewModel`) |
| API-DB | Cuenta, estructura funcional, columnas cifradas con clave de aplicación, encapsulaciones y metadata del Vault | Autorización por propietario, roles y cifrado de transporte | Una `APP_KEY` comprometida descifra el contenido de contacto, lugar, descripción y nota de cierre de **todas** las cuentas, no de una sola |
| Almacenamiento de objetos (S3) | Ciphertext y metadata del objeto (solo entradas migradas) | Cifrado autenticado de cliente, permisos y URL temporal | Exposición de metadata, borrado incompleto |
| Logs/Sentry | Eventos técnicos | Minimización nominal | No confirmado que excluya el contenido descrito en la sección 3 |

## 8. Búsqueda privada

La lista puede buscarse por nombre. El término de búsqueda no se envía a la API. Pero, a diferencia de lo que un modelo E2EE estricto requeriría, **sí existe un índice persistente en texto plano** (`vault_search_index`, FTS5) con nombre, profesión, lugar, descripción y nota de cierre. Borrar una nota localmente no siempre elimina la fila correspondiente de ese índice.

## 9. Versionado, sustitución y borrado

Cada nueva versión del Vault usa un ID remoto aleatorio distinto del ID funcional, y una versión anterior permanece disponible hasta confirmar que la nueva existe remotamente — con dos excepciones activas hoy:

- **Retry de URL expirada roto:** si la URL de subida expira, iOS reintenta esperando un `blobId` que la API no devuelve, y recibe en cambio una ruta temporal (`blobs/temp/...`) que no actualiza el objeto ya registrado.
- **Recovery agrupa y borra:** como se describe en la sección 4, la recuperación puede solicitar el borrado de entradas válidas de un mismo crush.

Al borrar un crush, iOS intenta borrar cada blob remoto y continúa si falla, luego elimina entradas y notas locales, y pide a la API el borrado del crush funcional. **La carpeta local de fotografías no se elimina en este flujo.**

## 10. Fotografías locales

- se seleccionan en el dispositivo;
- se decodifican y re-codifican sin conservar metadata EXIF, GPS o IPTC;
- se guardan con nombre aleatorio dentro de la carpeta local del crush;
- **no se excluyen del backup general del sistema operativo** (no se aplica `NSURLIsExcludedFromBackupKey`);
- no se envían a la API ni al almacenamiento remoto de MDL;
- no se cifran con la VMK;
- se eliminan solo por borrado individual manual; **no se eliminan al borrar el crush completo**.

## 11. Proveedores y fronteras

Ver la tabla verificada de proveedores en el [inventario maestro](02_MDL_Inventario_Maestro_Datos.md#12-proveedores-y-datos-recibidos). Dos hallazgos relevantes para seguridad:

- **Dos SDK de identidad activos en paralelo:** Firebase Auth y AWS Amplify con plugin Cognito Auth están ambos configurados en `AppDelegate`. No hay evidencia en este expediente de que ambos sean necesarios; deben revisarse antes de producción.
- **Plugin de almacenamiento de Amplify activo** junto al flujo de subida directa a S3 por URL firmada que usa el Vault; debe confirmarse qué datos, si alguno, pasan por ese canal.

## 12. Controles de seguridad verificados hoy

### Dispositivo

- Keychain con accesibilidad restringida y biometría para la VMK.
- Re-codificación de imágenes sin EXIF/GPS/IPTC.
- **Pendiente:** confirmación de que ninguna ruta persiste plaintext privado, términos de búsqueda o claves fuera del índice FTS5 ya identificado.

### API

- Autenticación Sanctum, autorización por propietario en recursos del Vault.
- URLs firmadas de 15 minutos para blobs.
- Rate limiting configurado por ruta (ver rutas `throttle:` en `/api/vault`).
- **Pendiente:** confirmación de que Sentry y los logs de aplicación excluyen los campos cifrados con clave de aplicación y el contenido en tránsito descrito en la sección 3.

### Infraestructura

- Cifrado en tránsito (HTTPS).
- Cifrado del proveedor en reposo (RDS, S3) como capa adicional a la aplicación.
- **Pendiente:** inventario de subencargados, procedimiento de incidentes documentado y ensayado, rotación de `APP_KEY`.

## 13. Riesgos que el estado actual no elimina

- La estructura funcional y las valoraciones son legibles por la API.
- El nombre del crush es legible por la API sin necesidad de descifrar nada.
- El resto del contenido de contacto, lugar, descripción y nota de cierre es descifrable por la API con una única `APP_KEY`; comprometer esa clave expone el contenido de todas las cuentas, no de una.
- El índice de búsqueda local guarda ese mismo contenido en texto plano en el dispositivo.
- La recuperación del Vault puede borrar entradas válidas de un mismo crush.
- La recuperación por frase de 24 palabras no está validada de punta a punta.
- Un bug de retry puede dejar un blob sin actualizar correctamente tras una URL expirada.
- Las fotografías dependen únicamente de controles locales, no se excluyen del backup del sistema y no se eliminan con el crush.
- Los datos sobre terceras personas plantean riesgos incluso cuando el contenido está cifrado.

## 14. Evidencia previa a producción

- decisión y fecha sobre si el nombre del crush se cifra antes de recibir tráfico real;
- validación de la recuperación por frase de 24 palabras en dispositivo limpio;
- corrección o mitigación documentada del bug de agrupación en recovery antes de que un usuario real lo dispare;
- confirmación de scrubbing de Sentry y logs de aplicación para los campos identificados;
- confirmación de qué SDK de identidad (Firebase, Amplify/Cognito) es el que realmente se usa en producción;
- inspección de `vault_search_index` en un dispositivo de prueba para confirmar el contenido en claro descrito acá;
- plan y fecha para retirar las columnas cifradas con clave de aplicación una vez el Vault esté completo.

Las cuestiones jurídicas y las medidas de reducción de riesgo a valorar están en la [guía de auditoría](05_MDL_Auditoria_Juridica.md).
