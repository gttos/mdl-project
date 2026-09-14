# Sign in with Apple para la beta

> **Estado:** API-APPLE-01 a 04 e iOS-APPLE-01 a 04 están implementadas; el alta Apple real, la capability/provisioning y la baja con Vault activo están validadas físicamente. La retención, el gate de disponibilidad y el diagnóstico JWK de API-APPLE-05 también están implementados. No queda una fase de código conocida: antes de cerrar permanecen el despliegue de API-APPLE-05, la matriz E2E restante (vínculo, revocación, relay y webhook reales) y los gates de configuración, Legal y disclosure público.
> **Prioridad:** requisito de salida de la beta iOS
> **Proyectos:** API e iOS
> **Última actualización:** 2026-09-10

## Objetivo

Permitir que una persona cree o abra su cuenta MDL mediante Sign in with Apple desde la app iOS, vinculando la identidad de Apple con el mismo `User`, la misma sesión y el mismo namespace/Vault que usa el acceso por email y contraseña.

## Baseline confirmado

- API ya valida Apple server-side, crea/vincula el mismo `User`, registra aceptaciones y emite el token Sanctum dentro de la operación atómica.
- iOS guarda el token de API, obtiene el perfil y activa el namespace/Vault antes de publicar la sesión autenticada; Apple reutiliza esa frontera.
- iOS ya declara la capability/entitlement de Sign in with Apple y ofrece el control nativo en Login y en la entrada de alta; la alta real con Private Relay fue validada en dispositivo.
- API conserva una ruta Google legacy sin flujo funcional; no participa ni define el contrato Apple.
- `users.password` admite `null`; las cuentas creadas con Apple pueden existir sin contraseña MDL.
- La verificación de email ya dispone de persistencia y endpoints API. Una alta Apple con email acreditado nace verificada, sin generar OTP.
- El régimen de cuenta provisional de Email Verification ya está desplegado: una alta Apple sólo puede salir de la transacción como cuenta verificada; nunca debe crear un plazo provisional ni alcanzar el gate de email.
- `AuthViewModel` publica una sesión Apple sólo después de activar el namespace/Vault y resolver el estado de verificación, sin ruta paralela de sesión.

## Alcance

- Botón nativo **Continuar con Apple** visible en la superficie de acceso no autenticada y en la entrada de registro.
- Autenticación nativa con `AuthenticationServices`, nonce criptográfico y envío de credenciales al backend.
- Verificación exclusiva en API de la identidad y autorización entregadas por Apple.
- Creación de un `User` nuevo con fecha de nacimiento, mayoría de edad y aceptaciones legales vigentes.
- Inicio de sesión de una identidad Apple ya vinculada.
- Vinculación explícita con una cuenta MDL existente, incluso cuando Apple usa Hide My Email o un email diferente.
- Integración con verificación de email, sesión de siete días, cambio de cuenta, Vault y App Lock.
- Revocación, baja de cuenta, limpieza de credenciales y eventos técnicos mínimos.
- Configuración operativa necesaria en Apple Developer, Private Email Relay y secretos de API.
- Cobertura automatizada y validación física end-to-end antes de la beta.

Fuera de alcance:

- Implementar Google o Facebook.
- Usar Firebase Auth como autoridad de sesión de MDL.
- Fusionar cuentas automáticamente sólo porque dos emails coinciden.
- Solicitar o almacenar el nombre de Apple; MDL no lo necesita para esta función.
- Cambiar la criptografía, recovery o reglas de dispositivo único del Vault.
- Web, Android, passkeys o una pantalla general de gestión de proveedores.
- Eliminar en esta iniciativa el `google_id`, Socialite o dependencias iOS legacy sin demostrar primero que no tienen consumidores.

## Reglas compartidas

### Identidad y cuenta canónica

- La cuenta canónica sigue siendo `User`; Sign in with Apple agrega un método de autenticación y nunca crea un segundo perfil/Vault para el mismo vínculo explícito.
- La identidad Apple se reconoce por el `sub` verificado y su issuer/audience, no por el email ni por el identificador que afirme el cliente.
- Un `sub` Apple sólo puede estar vinculado a un `User`, y un `User` sólo puede tener una identidad Apple activa.
- El email sigue siendo el email primario de la cuenta. Cambiarlo después no rompe la vinculación Apple.
- Una cuenta creada con Apple puede no tener contraseña. Puede establecer una mediante el flujo existente de recuperación por email; iOS no ofrece cambio de contraseña ni login biométrico basado en credenciales hasta que exista una contraseña MDL.

Esta iniciativa reemplaza durante la transición el requisito de contraseña de `PRODUCT.md` BR-3.01/BR-3.04 únicamente para el canal Apple. Fecha de nacimiento, mayoría de edad y aceptación legal siguen siendo obligatorias.

### Inicio de sesión

- iOS genera un nonce aleatorio por intento, envía su hash en la solicitud Apple y conserva el valor original sólo en memoria hasta entregarlo a API.
- iOS envía a API el identity token, el authorization code y el nonce; nunca decide por sí solo que la identidad es válida.
- API valida firma y rotación de claves, issuer, audience, expiración, nonce, authorization code, `sub` y `email_verified` antes de buscar o mutar una cuenta.
- Una identidad vinculada a una cuenta activa recibe el mismo token Sanctum de siete días que el login por contraseña.
- Cancelar el sheet de Apple no crea, vincula ni autentica una cuenta y no muestra un error alarmante.
- Una credencial inválida, expirada, reutilizada o destinada a otra app falla cerrada con un error saneado y sin sesión parcial.

### Registro con Apple

- Si el `sub` no está vinculado, API entrega una continuación opaca, de uso único y corta duración; no crea el usuario todavía.
- La elección inicial «Crear cuenta» o «Iniciar sesión» expresa sólo la intención de entrada; el botón Apple es neutral y API decide el resultado verificable. Si devuelve `registrationRequired`, iOS abre directamente el formulario de fecha de nacimiento y aceptaciones, sin una segunda pantalla de confirmación «Crear cuenta». Si devuelve `linkRequired`, iOS abre directamente la acreditación de la cuenta existente.
- Cuando una persona con una cuenta MDL preexistente necesite vincular un Apple ID cuyo email no permite a API clasificarla como `linkRequired`, el acceso manual a la acreditación permanece como acción secundaria dentro del formulario Apple de alta; no justifica una pantalla intermedia independiente ni permite merge automático.
- Crear la cuenta requiere fecha de nacimiento válida, edad mínima de 18 años y aceptación explícita de las versiones vigentes de términos y privacidad.
- API crea `User`, aceptaciones legales, identidad Apple y sesión en una única operación coherente. Un fallo no deja una cuenta o vinculación parcial.
- El email verificado por Apple se conserva como email primario, incluida una dirección Private Relay, y nace con `email_verified_at` confirmado; no se envía OTP.
- Si el token validado no contiene un email utilizable o el claim `email_verified` es falso/ausente, API falla cerrada: no emite continuación, sesión, OTP ni crea una cuenta. Apple entrega el email en el token de identidad y puede entregar una dirección Private Relay; MDL no intenta sustituir esa fuente por un email afirmado por iOS.
- Por minimización de datos, la continuación Apple no entrega a iOS el email ni la dirección Private Relay. Antes de confirmar el alta, iOS informa sólo que usará el email verificado proporcionado por Apple; no muestra ni persiste su valor. El email permanece dentro de los claims cifrados de API hasta que se complete el alta o vínculo.
- El nombre que Apple pueda entregar se ignora porque no forma parte de los datos necesarios para el alta MDL.

### Vinculación con una cuenta existente

- MDL nunca vincula automáticamente por coincidencia de email.
- Desde la continuación Apple, la persona puede indicar el email y contraseña de su cuenta MDL existente. API verifica ambos métodos antes de vincularlos.
- La vinculación conserva el UUID, email primario, verificación, aceptaciones, token/Vault y todos los datos de la cuenta existente.
- La dirección entregada por Apple puede diferir del email MDL; Hide My Email no obliga a crear una cuenta duplicada.
- Una contraseña incorrecta, cuenta inactiva, identidad ya vinculada o carrera concurrente no deja cambios parciales ni revela información adicional sobre otras cuentas.

### Email, sesión y Vault

- Para una cuenta Apple nueva, el claim verificado de Apple satisface el control inicial del email. La iniciativa de verificación de email debe omitir el envío automático de OTP cuando `email_verified_at` ya existe.
- La alta Apple fija `email_verified_at` y deja `provisional_expires_at` vacío **antes** de despachar el evento de registro; el listener de email no envía OTP y el scheduler de cuentas provisionales no puede seleccionarla.
- Vincular Apple a una cuenta existente no sustituye su email ni modifica su estado de verificación.
- Si la persona cambia después el email desde MDL, se aplica el flujo OTP canónico; la identidad Apple permanece por `sub`.
- El Private Email Relay debe estar configurado y probado antes de depender de emails operativos enviados a direcciones relay.
- Tras cualquier login/registro/vinculación, iOS guarda únicamente el token MDL, obtiene el perfil y activa el namespace/Vault correcto antes de publicar éxito.
- Un cambio de cuenta limpia el contexto en memoria y nunca adopta VMK, operaciones pendientes ni fotos de otro usuario.
- Identity token, authorization code, nonce y tokens Apple no se guardan en UserDefaults ni se imprimen en logs.

### Revocación y baja

- API conserva cifrado el refresh token mínimo necesario para validar/revocar la autorización de Apple; no lo devuelve al cliente.
- iOS observa el estado de la credencial Apple y su notificación de revocación. Un estado revocado/no encontrado elimina la sesión local y vuelve al acceso no autenticado.
- API valida las notificaciones server-to-server de Apple, desactiva la identidad afectada y revoca los tokens MDL activos. No elimina automáticamente el diario por una baja o revocación de la cuenta Apple.
- Una cuenta Apple-only puede recuperar acceso mediante una nueva autorización válida o establecer contraseña mediante el flujo de recuperación del email cuando éste siga disponible.
- La baja de MDL incluye revocar la autorización Apple, retirar la identidad federada y limpiar el identificador local, además del contrato general de baja/Vault. Un fallo temporal de Apple no restaura la cuenta MDL: tras cinco reintentos rápidos, el token cifrado queda sólo 30 días para un reintento manual autorizado y se elimina definitivamente al vencer ese plazo.
- Los digests de notificaciones server-to-server Apple se conservan 30 días para deduplicación; el último evento aplicable se conserva únicamente con la identidad Apple activa, para que una notificación tardía no revierta un estado más reciente.

## Escenarios canónicos

### GWT-APPLE-01 — Botón requerido en beta

**Dado** que la app está en el flujo no autenticado
**Cuando** muestra la entrada de login o registro
**Entonces** ofrece el botón nativo Continuar con Apple con tamaño, contraste, localización y accesibilidad equivalentes al acceso principal.

### GWT-APPLE-02 — Login de identidad vinculada

**Dado** un `sub` Apple válido vinculado a un usuario activo
**Cuando** API valida token, code y nonce
**Entonces** emite una sesión MDL de siete días para ese mismo `User` y iOS abre su mismo namespace/Vault.

### GWT-APPLE-03 — Alta nueva

**Dado** un `sub` Apple válido todavía no vinculado
**Cuando** la persona completa mayoría de edad y aceptaciones legales vigentes
**Entonces** se crean atómicamente un `User`, su identidad Apple y sus aceptaciones, se marca el email verificado y se inicia sesión.

### GWT-APPLE-04 — Registro incompleto

**Dado** un intento Apple válido sin fecha de nacimiento o aceptaciones válidas
**Cuando** se intenta completar el alta
**Entonces** API rechaza la creación y no persiste usuario, identidad, aceptación ni sesión parcial.

### GWT-APPLE-05 — Vincular cuenta existente

**Dado** un `sub` nuevo y una cuenta MDL existente, con email igual o diferente al de Apple
**Cuando** la persona acredita la contraseña de esa cuenta
**Entonces** API vincula Apple al `User` existente sin cambiar su email, verificación, UUID ni Vault.

### GWT-APPLE-06 — Sin merge automático

**Dado** que el email verificado de Apple coincide con una cuenta existente
**Cuando** todavía no se acreditó la contraseña MDL
**Entonces** no se vincula ni se emite sesión y se ofrece el flujo explícito de vinculación.

### GWT-APPLE-07 — Cancelación

**Dado** un sheet de autorización Apple visible
**Cuando** la persona cancela
**Entonces** la app permanece no autenticada, conserva el formulario y no muestra un fallo técnico.

### GWT-APPLE-08 — Credencial o replay inválido

**Dado** un token con firma/audience/nonce inválidos, expirado o un code/continuación ya consumidos
**Cuando** llega a API
**Entonces** se rechaza sin crear, vincular o emitir sesión y sin registrar secretos.

### GWT-APPLE-09 — Integración con email

**Dado** un alta Apple con `email_verified` válido
**Cuando** se dispara el comportamiento posterior al registro
**Entonces** la cuenta nace verificada y no recibe OTP; una vinculación posterior conserva el estado anterior.

### GWT-APPLE-10 — Aislamiento de cuenta

**Dado** que el dispositivo cambia de una cuenta A a otra B mediante Apple
**Cuando** iOS activa la nueva sesión
**Entonces** publica autenticación sólo después de activar el namespace B y no mezcla claves, entradas, pendientes ni fotos de A.

### GWT-APPLE-11 — Revocación

**Dado** que Apple revoca el consentimiento o elimina la credencial
**Cuando** iOS o API recibe una señal verificable
**Entonces** invalida el acceso Apple y las sesiones MDL afectadas sin interpretar ese evento como solicitud de borrar el diario.

### GWT-APPLE-12 — Baja MDL

**Dado** una cuenta con Apple vinculada
**Cuando** la baja MDL se confirma
**Entonces** se revoca la autorización Apple, se retiran identidad/tokens y se aplica el borrado general sin dejar material reutilizable.

## Datos, seguridad y Legal

El inventario técnico y la matriz de retención ya incorporan Sign in with Apple, incluida la política aprobada de revocaciones fallidas y notificaciones. Antes de activar públicamente la función, revisión humana/legal debe aprobar e incorporar en la documentación pública:

- `sub`/issuer/audience y email o Private Relay;
- estado de verificación/relay y timestamps de vinculación/revocación;
- authorization code, identity/access/refresh tokens y nonce según su vida técnica;
- finalidades, retención, cifrado, accesos, borrado y respuesta a derechos, incluida la ventana de 30 días para una revocación Apple fallida;
- Apple como destinatario/proveedor, región/transferencias y términos aplicables;
- política de privacidad, ficha de App Store Privacy y textos al usuario;
- efecto de notificaciones `email-disabled`, `email-enabled`, `consent-revoked` y `account-deleted`.

La política interna aprobada no sustituye la revisión humana/legal de la política pública, la ficha App Store Privacy, proveedor/transferencias ni textos al usuario. La activación productiva permanece bloqueada hasta esa revisión.

## Interacción entre proyectos

| Proyecto | Responsabilidad |
|---|---|
| Product | Congelar reglas, GWT, relación con email/Vault y gate Legal |
| API | Validar Apple, modelar identidad, crear/vincular el mismo `User`, emitir/revocar sesiones y manejar lifecycle |
| iOS | Presentar el botón nativo, obtener credenciales, completar registro/vinculación y preservar la frontera de sesión/Vault |

## Orden y fases transversales

1. **API-APPLE-01 — Persistencia y contrato:** identidad federada genérica, continuaciones y contrato HTTP estable; congela los estados que después consume iOS.
2. **API-APPLE-02 — Verificación Apple:** intercambio/validación server-side, nonce, JWKS y configuración segura con dobles; todavía sin credenciales reales ni despliegue.
3. **iOS-APPLE-01 — Capacidad y adaptador nativo:** puede empezar cuando API-APPLE-01 haya fijado el contrato; el portal/provisioning permanece como paso manual del Account Holder/Admin.
4. **API-APPLE-03 — Login/registro/vinculación:** transacciones, legal, email ya verificado, sesión y errores estables.
5. **iOS-APPLE-02/03 — Red, sesión y UI:** modelos/servicios, frontera `AuthViewModel`, botón y flujos de completar/vincular, sobre el contrato de API-APPLE-03.
6. **API/iOS-APPLE-04 — Lifecycle:** revocación, baja, relay, estado de credencial y limpieza local.
7. **PRODUCT/LEGAL-01 + API/iOS-APPLE-05 — Beta readiness:** completar disclosure Legal/App Store Privacy, configurar Apple/Private Relay/notificaciones, aplicar las retenciones aprobadas, realizar pruebas físicas y habilitar de forma compatible.

API debe estar desplegada y configurada antes de distribuir una build que muestre el botón. El botón no se habilita contra un backend sin soporte; una indisponibilidad temporal muestra retry y nunca crea una sesión local ficticia.

### Disponibilidad controlada durante beta

- La entrada de Apple se controla con un único flag de servidor, apagado por defecto. Mientras está apagado o la configuración está incompleta, las rutas de usuario `authorize`/`register`/`link` no inician ni mutan una sesión Apple; las notificaciones server-to-server continúan activas para no perder lifecycle de identidades ya existentes.
- iOS puede consultar una disponibilidad pública mínima que devuelve sólo un booleano `available`. No expone referencias de configuración, motivos de indisponibilidad, IDs, hosts, secretos ni el estado de notificaciones.
- El diagnóstico detallado de configuración y conectividad pertenece exclusivamente a una operación interna de API. No se convierte en un endpoint público ni en una pantalla de la app.

## Criterios de aceptación de la iniciativa

- [ ] Todos los GWT tienen una prueba API y/o iOS trazable.
- [ ] Alta, login y vinculación usan un único `User` y un único namespace/Vault.
- [ ] API valida code/token/nonce y nunca confía en email o `user` enviados sin firma.
- [ ] No existe merge automático por email ni duplicado silencioso con Hide My Email.
- [ ] Mayoría de edad y aceptaciones legales se aplican también al alta Apple.
- [ ] El email Apple válido nace verificado sin OTP; vincular no cambia el email existente.
- [ ] La sesión conserva duración, logout, restauración, App Lock y aislamiento actuales.
- [ ] Revocación y baja retiran acceso y tokens sin borrar el diario por una señal ambigua.
- [ ] No hay tokens, nonce, subject ni emails completos en logs o analítica.
- [ ] Private Email Relay y notificaciones Apple se prueban con configuración real.
- [ ] El botón cumple el HIG, está localizado en inglés/español y funciona con VoiceOver.
- [ ] Revisión Legal y App Store Privacy están aprobadas antes del rollout productivo.

## Estado resumido

| Proyecto | Estado | Próximo resultado |
|---|---|---|
| Product | Contrato Apple implementado; política interna de retención aprobada | disclosure Legal/App Store Privacy antes de activar públicamente |
| API | API-APPLE-01 a 04 implementadas y desplegadas; retención, disponibilidad segura (`05A`) y diagnóstico JWK opt-in (`05B`) implementados localmente | desplegar `05A/05B`, verificar configuración interna y ejecutar validación E2E/rollout con el flag controlado |
| iOS | iOS-APPLE-01 a 04 implementadas; alta Apple y baja con Vault activo validadas físicamente | vínculo manual y revocación de credencial en dispositivo; validaciones previas a beta |
| Transversal | código principal implementado; beta readiness pendiente | Private Relay/notificaciones y revocación reales, operación de la outbox y disclosure público |

## Planes locales

- [API](../../../../../api/docs/plans/active/sign-in-with-apple/README.md)
- [iOS](../../../../../ios/docs/plans/active/sign-in-with-apple/README.md)

## Referencias

- [`PRODUCT.md` §3](../../../PRODUCT.md#3-cuenta-autenticación-y-sesión)
- [`ARCHITECTURE.md` §2-4](../../../ARCHITECTURE.md#2-propiedad-de-datos)
- [Apple — Authenticating users with Sign in with Apple](https://developer.apple.com/documentation/signinwithapple/authenticating-users-with-sign-in-with-apple)
- [Apple — AuthenticationServices / credential state](https://developer.apple.com/documentation/authenticationservices/asauthorizationappleidprovider)
- [Apple — Token validation](https://developer.apple.com/documentation/signinwithapplerestapi/generate-and-validate-tokens)
- [Apple — Token revocation](https://developer.apple.com/documentation/signinwithapplerestapi/revoke-tokens)
- [Apple — Account deletion and revocation](https://developer.apple.com/documentation/technotes/tn3194-handling-account-deletions-and-revoking-tokens-for-sign-in-with-apple)
- [Apple — Sign in with Apple HIG](https://developer.apple.com/design/human-interface-guidelines/sign-in-with-apple/)

## Decisión abierta

1. **Revisión Legal previa al rollout:** el inventario y la retención técnica de Apple ya están fijados; faltan aprobación de privacy disclosures, ficha App Store Privacy, proveedor/transferencias y texto público. No bloquea las fases técnicas sin activación, pero sí cualquier rollout público.
