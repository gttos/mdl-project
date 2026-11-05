# Funciones futuras aprobadas para Jira

Material separado del catálogo vigente para su traslado al backlog. Las aprobaciones, restricciones y condiciones de activación siguientes conservan su alcance. Este documento no activa funciones ni inicia implementación.

<a id="adjetivos-negativos"></a>
## Adjetivos negativos

- **RP-ADJ-011.** Está aprobada la ampliación del catálogo compartido con adjetivos negativos para la autodescripción de la cuenta y la descripción de sus Crushes. Su lista final, definiciones ES/EN, pertenencia a bloques, identidades y presentación están pendientes de decisión en [Adjetivos negativos](plans/active/negative-adjectives/README.md). Las filas antiguas de una base de datos no quedan admitidas por esa aprobación: hasta cerrar y aplicar el catálogo ampliado siguen vigentes las 29 entradas de RP-ADJ-001 y las reglas actuales de selección, lectura y estadísticas. La ampliación no autoriza diagnósticos, inferencias automáticas ni agregados entre cuentas.
- **RP-ADJ-012.** La fuente única del catálogo compartido será la base de datos de API: opciones admitidas, IDs, bloques, orden, etiquetas y definiciones ES/EN. La app lo consultará mediante API. La feature de adjetivos negativos retirará `database/adjectives.json` y los catálogos alternativos que actúen como autoridad, sin fallback al archivo. Una caché del cliente sólo podrá ser una copia derivada del catálogo recibido. El traslado no activa todas las filas existentes: se publicará exclusivamente el catálogo aprobado en Product. Esta decisión está pendiente de implementación; no describe el origen del catálogo en el código vigente.

<a id="google"></a>
## Acceso Google en Android

- **RP-GGL-001.** «Continuar con Google» permite crear una cuenta MDL o entrar en una cuenta existente vinculada a Google.
- **RP-GGL-002.** «Continuar con Google» queda fuera de la beta activa iOS/API e iOS no lo muestra. Se reserva para una distribución Android posterior, en la que Android ofrecerá email y contraseña además de Google; la beta iOS ofrece email y contraseña además de Apple.
- **RP-GGL-003.** Cuando Android se habilite, obtendrá la autorización mediante el mecanismo nativo vigente de Google, pero sólo la API de MDL podrá validar la identidad y decidir si crea, vincula o abre una cuenta. Una sesión local de Google no equivale a una sesión MDL.
- **RP-GGL-004.** La API valida como mínimo firma, emisor, audiencia, vigencia y nonce de la credencial Google. Una credencial inválida, vencida, reutilizada o destinada a otro cliente se rechaza sin crear cuenta, vínculo ni sesión.
- **RP-GGL-005.** Una identidad Google sólo puede estar vinculada a una cuenta MDL. Una cuenta MDL sólo puede tener una identidad Google vinculada.
- **RP-GGL-006.** El alta con Google exige un email válido acreditado como verificado por Google, declarar la fecha completa de nacimiento, tener al menos 18 años y aceptar los términos y la política de privacidad vigentes. No exige crear una contraseña MDL.
- **RP-GGL-007.** MDL no solicita ni incorpora al perfil el nombre, la foto ni otros datos de perfil entregados por Google. Conserva únicamente la identidad, el email acreditado y la evidencia técnica necesaria para autenticar y gestionar el vínculo conforme al inventario de la cuenta.
- **RP-GGL-008.** El alta se completa cuando quedan confirmados atómicamente la cuenta, el acceso con Google, las aceptaciones legales y el inicio de sesión. Si falla alguna parte, no queda una cuenta creada parcialmente.
- **RP-GGL-009.** Cancelar la autorización Google conserva los datos introducidos en el formulario y no crea una cuenta, una vinculación ni una sesión. La cancelación no se presenta como un error.
- **RP-GGL-010.** Si el email acreditado por Google ya pertenece a una cuenta MDL, la coincidencia no concede acceso, no vincula y no fusiona cuentas automáticamente.
- **RP-GGL-011.** Vincular Google a una cuenta existente exige acreditar expresamente tanto la identidad Google como el acceso a la cuenta MDL y confirmar la vinculación. Puede iniciarse desde «Continuar con Google» acreditando email y contraseña MDL o desde los ajustes de una sesión MDL válida.
- **RP-GGL-012.** Si la identidad Google ya está vinculada a otra cuenta MDL, la operación se rechaza completa y ninguna de las dos cuentas cambia. MDL no fusiona, absorbe ni transfiere automáticamente cuentas.
- **RP-GGL-013.** Login, alta y vinculación Google utilizan el mismo `User`, la misma sesión MDL y el mismo namespace/Vault que el acceso por email y contraseña; no existe una cuenta ni un Vault paralelos por proveedor.
- **RP-GGL-014.** Una cuenta creada con Google que todavía no tenga contraseña puede añadir el acceso mediante email y contraseña desde los ajustes de una sesión MDL válida o mediante la recuperación enviada a su email vigente y verificado. La operación conserva Google, el identificador de la cuenta, el Vault y el diario y se rige por las mismas garantías atómicas de RP-APL-015, RP-APL-016, RP-APL-026 y RP-APL-032.
- **RP-GGL-015.** Una vez añadida la contraseña, la cuenta creada con Google puede acceder desde iOS mediante email y contraseña a la misma cuenta y al mismo Vault. iOS no necesita mostrar Google para permitir ese acceso.
- **RP-GGL-016.** Una cuenta MDL puede conservar Apple y Google como métodos de acceso sólo si cada identidad se vinculó expresamente a esa misma cuenta. La coincidencia de emails nunca crea el segundo vínculo.
- **RP-GGL-017.** Si se revoca la autorización Google o deja de ser válida, se impide el acceso con Google y se cierran las sesiones MDL afectadas. La cuenta, el Vault y el diario no se eliminan por ese hecho.
- **RP-GGL-018.** Si Google no está disponible, Android informa del problema y permite reintentar. No comunica que se ha iniciado sesión ni crea una sesión MDL ficticia.
- **RP-GGL-019.** El token de identidad, el nonce y cualquier credencial Google son transitorios: no se incorporan al perfil, al Vault, a logs ni a diagnósticos. Sólo se transportan y procesan durante la operación autorizada.
- **RP-GGL-020.** «Continuar con Google» no se habilita públicamente hasta que estén implementados y comprobados la validación server-side, la prevención de duplicados, la vinculación explícita, la revocación, la integración con sesión/Vault y los requisitos de configuración, marca, Google Play y revisión humana/legal.
- **RP-GGL-021.** Vincular Google conserva el email MDL y su estado de verificación, pero sólo puede hacerlo una cuenta MDL ya verificada. La vinculación desde «Continuar con Google» mediante email y contraseña responde `401 login_failed` para una provisional igual que para credenciales inválidas; la vinculación desde ajustes exige una sesión de cuenta válida y verificada. Ninguna ruta federada convierte una capacidad de registro en sesión ni completa el alta por email.

El acceso Google conserva las garantías compartidas de [Cuenta](rules/ACCOUNT.md): cifrado y minimización de todas las copias del email recibido del proveedor (RP-VLT-005 y RP-CUE-009), conservación del vínculo al cambiar email o contraseña (RP-MAIL-004 y RP-PWD-003), y cierre de sesiones tras cambiar la contraseña. Una cuenta creada con Apple puede acceder por email y contraseña desde Android una vez superados sus gates de distribución (RP-APL-034); conserva la misma cuenta y Vault. Estas referencias aplican a la distribución futura y no habilitan Google en la beta iOS/API.

<a id="agregados"></a>
## Estadísticas agregadas entre cuentas

- **RP-FUT-003.** La baja elimina la estructura y las contribuciones de la cuenta conforme a RP-BAJ-017; no conserva una fuente para agregados. Producto aprueba como función futura las estadísticas agregadas entre cuentas: incluirán, entre otros cortes que se definan, porcentajes de resultados relacionales para combinaciones de signos y patrones agrupados por país sobre países más presentes entre los Crushes, países más presentes entre los match y rasgos estructurales más frecuentes. Esta decisión funcional autoriza conservar dormantes el schema y los modelos reservados para esos resultados; no autoriza retener datos de cuentas eliminadas ni calcularlos, poblarlos o exponer resultados no vacíos. La activación requiere contrato de producto, población elegible, umbrales de agregación y supresión, actualización y retención, además de aprobación Legal. La infraestructura reservada no se elimina sólo por permanecer inactiva durante la beta.

<a id="almacenamiento"></a>
## Traslado del texto cifrado y reserva S3

- **RP-VLT-039.** Todos los bloques de texto cifrado del Vault se almacenan como contenido binario opaco en la misma base MySQL que su inventario, con asociación inequívoca al blob, revisión y propietario. Se conserva el cifrado en el dispositivo, sin descifrado del diario en API, nuevas claves ni modificación del payload privado. Tamaño, hash, cuota, revisión, idempotencia y autorización se verifican también en el nuevo almacenamiento. Los listados y respuestas funcionales no incluyen los bytes privados cifrados por defecto; su transferencia utiliza el canal autorizado del Vault. La transición desde S3 se valida antes de declarar implementado este destino.
- **RP-VLT-040.** Se conserva el código de conexión y los adaptadores/métodos de S3 necesarios para una futura función de fotografías remotas, expresamente documentados y desconectados del flujo de textos. Es una reserva técnica autorizada, no una función activa ni un fallback de escritura o lectura del diario. No se mantienen dos destinos activos para los textos ni se elimina la infraestructura S3 por esta decisión. No se habilitan subida, descarga, backup o recuperación remota de fotos: RP-FOT-001..012 siguen vigentes y cualquier función futura requiere diseño, tratamiento y aprobación propios.
- **RP-VLT-041.** El cliente transfiere los bloques de texto cifrado mediante HTTP autenticado contra la API, con validación de propietario, identidad, tamaño e integridad y las garantías durables vigentes. La implementación sustituye el transporte S3 de texto y coordina API y clientes antes de distribuir el nuevo contrato; no simula campos o firmas de S3 para mantener artificialmente el protocolo anterior. No cambia pantallas, formato criptográfico, recovery o fotos locales por el mero cambio de transporte. Las reservas y reintentos de RP-VLT-031 conservan su identidad y plazo.

<a id="personalidad"></a>
## Tests de personalidad

- **RP-CON-005.** El progreso de un test de personalidad sin terminar —respuestas guardadas y paso alcanzado— se conserva para poder reanudarlo hasta que se confirme su finalización o se cierre la cuenta, lo que ocurra primero. No caduca por antigüedad ni por falta de actualizaciones. La finalización retira el progreso sólo después de guardar correctamente el resultado; esa retirada no elimina el resultado final. El cierre de cuenta elimina el progreso pendiente como parte de la baja. Esta decisión conserva la función para su uso futuro y no autoriza habilitarla ahora en la beta ni define la retención de sus resultados finales.

<a id="android"></a>
## Lanzamiento Android y reconciliación con iOS

- **RP-LAN-005.** «Continuar con Google» queda fuera de la beta activa iOS/API. Antes de una futura distribución Android debe comprobarse en conjunto validación server-side y anti-replay, vinculación explícita sin duplicados, revocación, sesión/Vault, configuración operativa, requisitos de marca y Google Play, disclosure y revisión humana/legal; esos gates no bloquean la beta iOS/API.
- **RP-LAN-008.** Android se desarrolla durante la beta iOS/API para un lanzamiento conjunto de iOS y Android previsto en enero de 2027, sin día de publicación fijado. Su alcance incluye los mismos recorridos funcionales y presentación de iOS, incluida la demo nativa de RP-DEMO-001..014, traducidos a componentes Android. La exclusión de Android de la beta es de distribución y no de desarrollo. Se conservan los métodos de acceso por plataforma de RP-LAN-004/005 y las excepciones expresas del catálogo; las menciones a Face ID/Touch ID se adaptan a la biometría nativa Android sin cambiar su función. Antes del lanzamiento deben estar implementadas y validadas la paridad, la interoperabilidad del Vault y las garantías propias de ambas plataformas. La fecha objetivo no sustituye esos gates ni habilita publicación, nuevos tratamientos o cambios jurídicos. Para la reconstrucción inicial, la referencia funcional y visual es el SHA iOS fijo identificado en ese plan: primero se clona íntegramente ese corte en Android y después se incorporan los incrementos de la versión futura de iOS. Las definiciones de Product que describen esos incrementos posteriores —incluidos nuevos textos/explicaciones de situaciones y niveles/respuestas de experiencias— conservan su validez para la versión futura, pero no se exigen como criterio de salida de las fases del clonado fijo. Su reconciliación explícita se realiza después de completar el clonado, antes de validar y distribuir el corte final. Esta secuencia no autoriza contratos HTTP alternativos, fallbacks de compatibilidad ni cambios en Legal, seguridad o los gates de distribución. La ejecución se coordina en el [plan de paridad Android](plans/active/android-ios-parity/README.md).

<a id="seguridad-publica"></a>
## Evaluación del lanzamiento público

- **RP-LAN-006.** La beta 0.2 no queda bloqueada por la evaluación adversarial del lanzamiento público posterior. Antes de ese lanzamiento, MDL debe completar una evaluación autorizada del artefacto iOS firmado y de la API desde la perspectiva de un cliente sin acceso al código, resolver los hallazgos que superen el umbral aprobado y repetir la misma evaluación sobre el candidato de lanzamiento. La política de autenticidad cliente-API y cualquier tratamiento de señales de dispositivo o riesgo deben quedar definidos, probados y, cuando corresponda, aprobados por Legal. La atestación, la ofuscación o la detección de manipulación son controles complementarios: nunca sustituyen sesión, propiedad, autorización, límites de abuso, cifrado ni validación server-side.

## Entrada de demo en la distribución pública

La entrada informativa de demo de RP-DEMO-012 se ofrece después de resolver la protección propia en la etapa pública. Su activación conserva los controles de distribución y telemetría.

## Condiciones de alcance conservadas

- La ampliación de accesibilidad y texto aumentado requiere una decisión específica; se conserva el soporte existente.
- El registro de bajas y sus copias permanecen sujetos a DEC-14 antes de cualquier uso real.
- El catálogo negativo, sus límites e integración estadística permanecen pendientes de las decisiones NEG-01.

## Planificación y decisiones relacionadas

- [Decisiones pendientes](DECISIONES_PARA_JIRA.md).
- [Adjetivos negativos](plans/active/negative-adjectives/README.md).
- [Google](plans/active/sign-in-with-google/README.md).
- [Paridad Android](plans/active/android-ios-parity/README.md).
- [Almacenamiento y restauración](plans/active/backup-restore-safety/README.md).

## Registro independiente de bajas

El comportamiento y los límites del registro se definen en [RP-BCK-003](rules/DATA_LIFECYCLE.md#restauracion-segura). Su contrato técnico aprobado y pendiente de implementación se mantiene en el [plan API de restauración](../../api/docs/plans/active/backup-restore-safety/README.md#dynamodb-independiente); su uso real depende de [DEC-14](DECISIONES_PARA_JIRA.md#dec-14).
