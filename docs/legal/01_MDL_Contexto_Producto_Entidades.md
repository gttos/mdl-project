# 1. Contexto del producto y sus entidades

> Este documento explica MDL en lenguaje de producto, tal como funciona hoy. Los campos y ubicaciones exactos están en el [inventario maestro](02_MDL_Inventario_Maestro_Datos.md).

## 1. Qué es MDL

MDL es una aplicación iOS de uso personal para organizar información sobre vínculos afectivos. Permite registrar personas de interés, experiencias compartidas, evolución de relaciones, valoraciones subjetivas, fotografías locales y métricas derivadas del historial.

No es una red social: la persona descrita no crea necesariamente una cuenta, no recibe un perfil público y no participa en el registro. El contenido lo introduce y consulta el usuario titular de la cuenta.

## 2. Qué intenta resolver

El producto busca que el usuario pueda:

1. recordar quién es una persona y datos que considera relevantes;
2. registrar cuándo y cómo fue una experiencia;
3. seguir el comienzo, evolución o cierre de una relación;
4. consultar todos esos eventos en un timeline;
5. obtener métricas personales basadas en la información estructurada introducida.

## 3. Personas afectadas

| Persona | Relación con MDL | Datos principales |
|---|---|---|
| Usuario | Crea una cuenta y usa la aplicación | Identidad, contacto, autenticación, uso, registros, valoraciones y métricas |
| Crush o tercera persona | Persona descrita por el usuario; normalmente no tiene cuenta | Perfil, características, contacto, fechas, estados, valoraciones, experiencias y relaciones |
| Personal autorizado | Acceso técnico a la base de datos o a la infraestructura de producción | Puede leer directamente las columnas cifradas con la clave de aplicación descrita en la sección 6; no existe hoy un panel de administración separado |
| Proveedores | Prestan infraestructura o servicios auxiliares | Ver la lista verificada en el [inventario maestro](02_MDL_Inventario_Maestro_Datos.md#12-proveedores-y-datos-recibidos) |

"Crush" es el nombre de producto para la ficha privada de una persona que interesa al usuario. No implica que esa persona conozca o acepte la creación de la ficha.

## 4. Recorrido principal del usuario hoy

### 4.1 Cuenta y Vault

El usuario se registra con email, contraseña y fecha de nacimiento, o mediante Google OAuth. Por separado, puede configurar un Vault: un contenedor criptográfico en el que la app cifra contenido en el dispositivo con una clave (VMK) que el servidor nunca recibe en claro.

El Vault no es parte obligatoria del alta de cuenta ni sustituye hoy a las columnas funcionales descritas en 4.2: ambos mecanismos conviven para el mismo contenido.

### 4.2 Crear un crush

El usuario crea una ficha de la persona con un único formulario. Hoy ese formulario envía a la API, en la misma operación:

- **estructura funcional en texto plano:** ID, edad, país, género, fecha en que se conocieron, estado, características seleccionadas y valoraciones;
- **contenido de contacto:** nombre (se guarda en texto plano en la base de datos), profesión, teléfono o mensajería, redes sociales y fecha de nacimiento. Salvo el nombre, estos campos se cifran al guardarse con la clave de aplicación (`APP_KEY`) y el backend los descifra automáticamente cada vez que arma una respuesta;
- **fotografías:** archivos sanitizados que permanecen exclusivamente en el dispositivo.

Además, si el usuario tiene el Vault activo, iOS construye una segunda copia cifrada con su VMK y la sincroniza como blob independiente. Esa segunda copia no reemplaza a la primera: la columna funcional sigue existiendo, legible por el backend, salvo que un proceso posterior la elimine (hoy no ocurre; ver sección 6).

### 4.3 Crear una experiencia

Una experiencia es un evento asociado a un crush. La API conserva su ID, fecha, asociación y nivel de satisfacción en columnas legibles. El lugar y la descripción viajan en el mismo request HTTP y se guardan cifrados con la clave de aplicación en las columnas `place` y `description`. Si el Vault está activo, además se cifra una copia con VMK y se sincroniza como blob.

### 4.4 Crear o actualizar una relación

Una relación representa un periodo o estado relacional con un crush. La API conserva fechas, estado, asociación y, cuando corresponda, vínculo con una experiencia, en columnas legibles. El texto de cierre viaja en el mismo request y se guarda cifrado con la clave de aplicación en la columna `closure_note`. Si el Vault está activo, además se cifra una copia con VMK y se sincroniza como blob.

### 4.5 Timeline, lista y métricas

La API entrega la estructura ordenada de crushes, experiencias y relaciones, incluyendo el contenido de las columnas cifradas con clave de aplicación ya descifrado cuando el Resource correspondiente lo incluye. iOS puede además superponer contenido proveniente del Vault descifrado localmente cuando esa entrada ya fue migrada. Las métricas se calculan a partir de datos funcionales legibles por la API.

La búsqueda por nombre en la lista de crushes se realiza localmente. iOS mantiene además un índice local de texto (FTS5) que guarda nombre, profesión, lugar, descripción y nota de cierre **en texto plano** en el dispositivo, independiente de las tablas cifradas del Vault. El término de búsqueda no se envía a la API.

### 4.6 Borrado

El usuario puede borrar una fotografía, una entidad concreta, un crush con sus entidades asociadas o solicitar la baja de cuenta. Al borrar un crush, la API elimina (soft delete) su fila y las de sus experiencias y relaciones, incluyendo las columnas cifradas con clave de aplicación. Si existen blobs del Vault, iOS intenta borrarlos remotamente y continúa aunque falle. La carpeta local de fotografías del crush **no se elimina** en este flujo y permanece en el dispositivo.

## 5. Las tres entidades prioritarias

| Entidad | Qué representa | Datos en columnas funcionales (API-DB) | Contenido también presente en Vault, solo para entradas migradas |
|---|---|---|---|
| Crush | Ficha privada de una persona | IDs, edad, país, género, estados, fechas, características, valoraciones, nombre (texto plano), email/teléfono/redes (cifrados con clave de aplicación) | Nombre, profesión, contacto, redes y fecha de nacimiento |
| Experiencia | Evento vivido o registrado con un crush | IDs, asociación, fecha, satisfacción, lugar y descripción (cifrados con clave de aplicación) | Lugar y descripción |
| Relación | Periodo o estado relacional con un crush | IDs, asociación, estado, fechas, vínculo con experiencia, nota de cierre (cifrada con clave de aplicación) | Nota de cierre |

La columna "cifrados con clave de aplicación" no es una frontera de seguridad equivalente al Vault: una sola clave de aplicación protege el contenido de todas las cuentas, y el backend descifra ese contenido de forma rutinaria al servir respuestas.

## 6. Dónde vive la información hoy

| Capa | Función | Qué puede leer |
|---|---|---|
| App iOS | Interfaz, cifrado/descifrado del Vault, búsqueda local y fotografías | Datos funcionales, contenido de columnas cifradas con clave de aplicación (vía API) y, mientras el Vault está desbloqueado, contenido descifrado con VMK |
| Keychain de iOS | Custodia de la VMK y claves locales | Material criptográfico bajo controles del sistema |
| Base local de iOS | Vault SQLite, cola y **índice de búsqueda en texto plano** | Ciphertext y metadata del Vault; el índice FTS5 guarda texto legible |
| API y base relacional (MySQL/RDS) | Cuenta, estructura funcional, columnas cifradas con clave de aplicación y metadata del Vault | Todo el contenido funcional; descifra de forma transparente `Crush.email/whatsapp*/instagram/facebook/snapchat/tiktok`, `Experience.description/place`, `Relationship.closure_note` y `User.phone`. No lee el ciphertext del Vault |
| Almacenamiento remoto de objetos (S3) | Conservación de blobs del Vault | Ciphertext y metadata del objeto, solo para entradas ya migradas al Vault |
| Proveedores auxiliares | Autenticación federada, correo, observabilidad, distribución | Ver el [inventario maestro](02_MDL_Inventario_Maestro_Datos.md#12-proveedores-y-datos-recibidos) |

## 7. Flujo conceptual de datos hoy

1. El usuario introduce datos en iOS en un único formulario por entidad.
2. iOS envía todos los campos —funcionales y de contacto/lugar/descripción/nota— en el mismo request HTTP a la API.
3. La API persiste los campos funcionales en texto plano y cifra con la clave de aplicación los campos de contacto, lugar, descripción y nota de cierre antes de guardarlos.
4. Si el usuario tiene el Vault activo, iOS cifra en paralelo una copia del mismo contenido con su VMK y la sincroniza como blob independiente hacia S3, vía la API.
5. Para leer, iOS obtiene desde la API la estructura funcional junto con el contenido de las columnas cifradas con clave de aplicación ya descifrado; si la entrada también existe en el Vault, iOS puede optar por mostrar la versión descifrada localmente en su lugar.

## 8. Puntos que la asesoría debe entender desde el inicio

- Una parte relevante de los datos describe a terceras personas que no interactúan con MDL.
- El nombre del crush se guarda en texto plano en la base de datos de producción.
- El resto del contenido de contacto, lugar, descripción y nota de cierre está cifrado con una única clave de aplicación que el backend usa para descifrar automáticamente; no es cifrado de extremo a extremo.
- El texto privado también viaja por HTTP en el mismo request que los campos funcionales, incluso cuando el usuario tiene el Vault activo.
- El Vault E2EE existe y está parcialmente activo, pero no sustituye a las columnas anteriores: ambos coexisten para el mismo contenido.
- Las fotografías son locales, no se excluyen del backup del sistema operativo y no se borran automáticamente al borrar el crush.
- Las métricas son individuales y se basan en datos identificables; no deben presentarse como anónimas.

## 9. Preguntas de contexto para la asesoría

1. ¿Cómo debe calificarse el papel de MDL respecto de los datos que un usuario introduce sobre una tercera persona?
2. ¿Qué deberes de información existen respecto de esas terceras personas y qué excepciones podrían resultar aplicables?
3. ¿El uso de una única clave simétrica de aplicación para cifrar en reposo el contenido de todas las cuentas constituye una medida de seguridad suficiente para el tipo de datos involucrados, mientras el Vault E2EE no reemplace ese mecanismo?
4. ¿Qué edad mínima y mecanismo proporcionado de verificación deben aplicarse en cada país de lanzamiento?
5. ¿Debe suspenderse el paso a producción hasta que el nombre del crush y el resto del contenido descrito se cifren, o es aceptable lanzar con el estado descrito en este expediente sujeto a un plan de corrección con fecha?
