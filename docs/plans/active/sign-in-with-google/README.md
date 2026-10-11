# Sign in with Google para una distribución Android futura

> **Definición funcional vigente:** [Acceso con Google](../../../rules/ACCOUNT.md#google), [Acceso con Apple](../../../rules/ACCOUNT.md#apple) y [Condiciones de lanzamiento](../../../PRODUCT.md#lanzamiento). La beta activa incluye sólo iOS y API: iOS ofrece email/contraseña y Apple. Android y Google se preparan para una distribución posterior, con gates propios que no bloquean la beta activa. Esta decisión no habilita por sí sola distribución, credenciales reales ni despliegue.

> **Estado:** `AND-GOOGLE-01`, `API-GOOGLE-01` y `API-GOOGLE-02` están aceptadas con evidencia automatizada. La implementación presentada de `AND-GOOGLE-02` no está aceptada: el acceso público consume correctamente `authorize → login`, pero la vinculación desde Ajustes llama también a `login` y entrega a `link-from-session` la continuación de segunda etapa. El contrato API exige que esa entrada autenticada conserve y consuma directamente la continuación `google_identity_verified` de `authorize`; nunca debe autenticar antes otra cuenta ni usar `google_identity_pending_link`. La corrección y sus pruebas pertenecen todavía a `AND-GOOGLE-02`. `API-GOOGLE-03` y los gates de distribución siguen pendientes.
> **Prioridad:** avance Android paralelo, independiente de los planes activos de iOS
> **Proyectos:** Product, API y Android
> **Última actualización:** 2026-10-07

## Objetivo

Permitir que una persona cree o abra su cuenta MDL mediante Google desde Android, usando el mismo `User`, la misma sesión y el mismo namespace/Vault que el acceso por email y contraseña, sin habilitar Google en iOS ni depender de las fases activas de Sign in with Apple.

## Alcance

- Botón **Continuar con Google** sólo en Android.
- Autorización Android mediante Credential Manager y credenciales Google vigentes, con nonce criptográfico.
- Validación exclusiva en API de la credencial Google.
- Alta con email acreditado por Google, fecha completa de nacimiento, mayoría de edad y aceptaciones legales vigentes.
- Login de una identidad ya vinculada y vinculación explícita con una cuenta MDL existente.
- Primera contraseña para una cuenta creada con Google, conservando la misma cuenta y Vault.
- Integración con la sesión MDL de siete días, bootstrap, cambio de cuenta, Vault y App Lock.
- Revocación, baja, limpieza de credenciales y eventos técnicos mínimos.
- Cobertura automatizada, validación física y gates de configuración, Google Play y Legal antes de mostrar la función públicamente.

Fuera de alcance:

- Mostrar Google en iOS o Apple en Android.
- Usar Firebase Auth o una sesión Google como autoridad de sesión de MDL.
- Fusionar cuentas automáticamente por coincidencia de email.
- Importar nombre, foto u otros datos de perfil Google.
- Cambiar la criptografía, recovery o reglas de dispositivo único del Vault.
- Implementar estadísticas o Relationship como parte de esta iniciativa.
- Configurar credenciales reales, desplegar API o publicar una build sin autorización propia.

## Decisiones y contrato compartido

1. Android conserva email/contraseña y añade Google; iOS conserva email/contraseña y Apple.
2. Android usa Credential Manager, no el cliente Google Sign-In legacy ni Firebase Auth como autoridad.
3. Android sólo obtiene una credencial y un nonce; la API valida identidad, emisor, audiencia, vigencia, nonce y replay antes de emitir una sesión MDL.
4. Cada identidad Google y cada cuenta MDL admiten como máximo un vínculo Google.
5. La coincidencia de emails nunca autentica, vincula ni fusiona cuentas.
6. Toda vinculación es explícita y conserva el mismo `User`, UUID, namespace/Vault y diario.
7. El token de identidad y el nonce son transitorios y no se guardan ni registran.
8. Una cuenta creada con Google puede añadir email/contraseña y acceder desde iOS sin que iOS muestre Google.
9. La primera fase Android permanece desconectada de UI productiva, red, sesión y Vault hasta que API congele el contrato HTTP.

## Escenarios de aceptación

### GWT-GOOGLE-01 — Separación por plataforma

**Dado** el flujo no autenticado de Android
**Cuando** se habilita la función en una distribución Android futura
**Entonces** ofrece email/contraseña y Google, no Apple; iOS ofrece email/contraseña y Apple, no Google.

### GWT-GOOGLE-02 — Login de identidad vinculada

**Dado** una identidad Google válida vinculada a un usuario activo
**Cuando** API valida token, audiencia, vigencia, nonce y replay
**Entonces** emite una sesión MDL para ese mismo `User` y Android abre su mismo namespace/Vault.

### GWT-GOOGLE-03 — Alta nueva

**Dado** una identidad Google válida no vinculada con email acreditado
**Cuando** la persona declara una fecha que acredita mayoría de edad y acepta las versiones legales vigentes
**Entonces** se crean atómicamente el `User`, la identidad Google y sus aceptaciones y se inicia sesión sin contraseña ni OTP MDL.

### GWT-GOOGLE-04 — Sin merge automático

**Dado** que el email Google coincide con una cuenta MDL existente
**Cuando** todavía no se acreditó esa cuenta
**Entonces** no se vincula ni se emite sesión y se ofrece el recorrido explícito de vinculación.

### GWT-GOOGLE-05 — Vinculación explícita

**Dado** una cuenta MDL existente y una identidad Google todavía libre
**Cuando** la persona acredita ambos accesos y confirma la vinculación
**Entonces** Google se añade al mismo `User` sin cambiar email, UUID, Vault ni diario.

### GWT-GOOGLE-06 — Cancelación y fallo

**Dado** el selector de Google abierto
**Cuando** la persona cancela o la credencial falla
**Entonces** Android permanece no autenticado, conserva el formulario y diferencia cancelación neutral de fallo reintentable sin crear estado durable.

### GWT-GOOGLE-07 — Credencial inválida o replay

**Dado** un token con firma, emisor, audiencia, vigencia o nonce inválidos, o una continuación ya consumida
**Cuando** llega a API
**Entonces** se rechaza sin crear, vincular o emitir sesión y sin registrar credenciales.

### GWT-GOOGLE-08 — Aislamiento de cuenta

**Dado** que el dispositivo cambia de una cuenta A a otra B mediante Google
**Cuando** Android activa la nueva sesión
**Entonces** publica autenticación sólo después de activar el namespace B y no mezcla claves, entradas, pendientes ni fotos de A.

### GWT-GOOGLE-09 — Acceso entre plataformas

**Dado** una cuenta creada con Google
**Cuando** añade y verifica su primera contraseña MDL
**Entonces** puede acceder por email/contraseña desde iOS al mismo `User` y Vault sin retirar Google.

### GWT-GOOGLE-10 — Revocación o indisponibilidad

**Dado** que Google revoca la autorización o no está disponible
**Cuando** MDL recibe una señal verificable o falla la autorización
**Entonces** impide el acceso Google y revoca las sesiones afectadas cuando corresponda, pero no borra el diario; una indisponibilidad permite reintentar y nunca simula una sesión.

## Interacción entre proyectos

| Proyecto | Responsabilidad |
|---|---|
| Product | Reglas, GWT, separación de plataforma y gates de lanzamiento |
| API | Verificar Google, corregir la ruta legacy, crear/vincular el mismo `User`, emitir/revocar sesiones y manejar lifecycle |
| Android | Credential Manager, credencial/nonce, flujos de alta/vínculo y frontera de sesión/Vault |
| iOS | No implementa Google; sólo debe poder abrir por email/contraseña la misma cuenta cuando ese método exista |

## Orden y fases transversales

1. **AND-GOOGLE-01 — Adaptador nativo aislado:** fase aceptada. Credential Manager, configuración placeholder, nonce y resultados tipados están integrados sin consumidor productivo, red, sesión ni Vault. La suite JVM quedó en 463/463; `GoogleCredentialResponseMapperInstrumentedTest` pasó 2/2 en el AVD Pixel_9a/API 37, sin fallos, errores ni omitidos.
2. **API-GOOGLE-01 — Contrato y verificador:** fase aceptada. La ruta legacy se retiró; la verificación server-side emite una continuación de un solo uso acotada a Google. El consumo exige proveedor y propósito para impedir que una continuación Google sea aceptada por un flujo Apple. Sin credenciales reales ni despliegue.
3. **API-GOOGLE-02 — Login, alta y vínculo:** fase aceptada con evidencia automatizada. Incluye las dos entradas de vinculación de RP-GGL-011 y la primera contraseña por Ajustes o recuperación de RP-GGL-014, sin activación pública.
4. **AND-GOOGLE-02 — Red, sesión y UI:** implementación en corrección. El recorrido público consume `authorize → login`; la entrada desde Ajustes debe consumir `authorize → link-from-session` directamente y demostrar que nunca llama a `login` ni adopta una sesión ajena.
5. **API/AND-GOOGLE-03 — Lifecycle:** revocación, baja, limpieza local y recuperación/primera contraseña.
6. **PRODUCT/LEGAL + API/AND-GOOGLE-04 — Beta readiness:** disclosures, configuración Google Cloud/Play, marca, pruebas físicas y E2E, despliegue compatible y habilitación controlada.

`AND-GOOGLE-01` no depende de iOS ni del contrato final de Relationship. `AND-GOOGLE-02` no comienza hasta que API haya congelado sus estados y payloads. Una build pública nunca muestra el botón contra un backend sin soporte.

## Estado técnico vigente

- Android ya implementa email/contraseña, registro, verificación, recuperación y el bootstrap que separa sesión inexistente, email provisional, perfil, onboarding/recovery del Vault y diario.
- Android contiene el adaptador `GoogleCredentialGateway`/`AndroidGoogleCredentialGateway`, la red y las superficies de acceso y Ajustes detrás de un client ID todavía placeholder. `AND-GOOGLE-02` no está aceptada porque `GoogleAccountLinkViewModel` usa `authorize → login → link-from-session`; debe preservar la continuación de `authorize` y llamar directamente a `link-from-session`, sin ejecutar `login` desde una sesión abierta.
- API expone verificación, disponibilidad, login, alta y vinculación Google; la ruta callback legacy fue retirada. La identidad verificada pasa por continuaciones de un solo uso antes de crear, vincular o autenticar una cuenta.
- API reutiliza infraestructura federada compartida y restringe cada continuación al proveedor y propósito esperados; los flujos Apple conservan su comportamiento válido.
- Estadísticas Android todavía consume el modelo Relationship anterior; su sustitución pertenece a `AND-GF-02` sobre el contrato canónico ya cerrado.

## Gates de distribución y Legal

La implementación técnica puede avanzar sin modificar `docs/legal/`. La activación pública queda bloqueada hasta una revisión humana/legal y de Google Play que determine disclosures, proveedor/destinatario, datos tratados, finalidades, retención, transferencias, revocación, textos al usuario y ficha Data safety. También requiere credenciales reales, fingerprints, audiencias y configuración de producción autorizadas, además de E2E físico contra la API desplegada.

## Próximo resultado permitido

El siguiente resultado permitido es corregir y volver a presentar `AND-GOOGLE-02`: la vinculación desde Ajustes usa la continuación de `authorize` en `link-from-session`, no llama a `login`, no recibe ni descarta un token de otra cuenta y cuenta con pruebas que fallen si reaparece cualquiera de esas rutas. No se avanza a `AND-GOOGLE-03`, no se muestra públicamente el botón y no se despliega ni configura ningún secreto. Después de aceptar la corrección seguirán la validación instrumentada/física, lifecycle y los gates operativos y humano/legal.

## Planes locales

- [API](../../../../../api/docs/plans/active/sign-in-with-google/README.md)
- [Android](../../../../../android/docs/plans/active/sign-in-with-google/README.md)

## Referencias

- [Cuenta](../../../rules/ACCOUNT.md)
- [Producto](../../../PRODUCT.md)
- [Arquitectura](../../../ARCHITECTURE.md)
- [Sign in with Apple](../sign-in-with-apple/README.md)
- [Android parity](../android-parity/README.md)
