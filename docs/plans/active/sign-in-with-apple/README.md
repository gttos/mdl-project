# Sign in with Apple para la beta

> **Definición funcional vigente:** [Acceso con Apple](../../../rules/ACCOUNT.md#apple). Están aprobados los recorridos de [cuenta Apple a email y contraseña](../../../rules/ACCOUNT.md#apple-a-email) y de [cuenta con email y contraseña a Apple](../../../rules/ACCOUNT.md#email-a-apple). Su implementación completa está pendiente; la evidencia técnica de partes existentes no acredita estos recorridos completos. `DEC-01` queda limitada al tratamiento de dos cuentas MDL distintas ya creadas. Esta revisión documental no inicia una fase de código ni acepta implementación.

> **Estado:** API-APPLE-01 a 04, iOS-APPLE-01/02/04 e iOS-APPLE-03 en su ruta `linkRequired` están implementadas; el alta Apple real, la capability/provisioning y la baja con Vault activo están validadas físicamente. La retención, el gate de disponibilidad y el diagnóstico JWK de API-APPLE-05 también están implementados. `IOS-APPLE-03R` incorporó la acción manual de vínculo para Private Relay/email distinto y quedó validada por build y pruebas dirigidas. Antes de la matriz E2E anti-duplicación falta comprobarla en dispositivo. Después permanecen el despliegue de API-APPLE-05, revocación, relay y webhook reales, y los gates de configuración, Legal y disclosure público.
> **Prioridad:** requisito de salida de la beta iOS
> **Proyectos:** API e iOS
> **Última actualización:** 2026-09-14

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
- Creación de un `User` nuevo con los datos de nacimiento definidos en `RP-CUE-001`/`RP-APL-005`, mayoría de edad y aceptaciones legales vigentes; la adaptación al año se ejecuta en [Información del usuario](../user-information/README.md).
- Inicio de sesión de una identidad Apple ya vinculada.
- Vinculación explícita con una cuenta MDL existente, incluso cuando Apple usa Hide My Email o un email diferente.
- Incorporación del acceso por email y contraseña a una cuenta creada con Apple, conforme al recorrido canónico, conservando la misma cuenta y Vault.
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

La fuente única es [Acceso con Apple](../../../rules/ACCOUNT.md#apple), `RP-APL-001..031`, junto con [Cuenta](../../../rules/ACCOUNT.md), [Baja](../../../rules/DATA_LIFECYCLE.md#baja) y [Retención](../../../rules/DATA_LIFECYCLE.md#plazos).

| Tema | Referencia |
|---|---|
| Identidad, validación y unicidad | RP-APL-001..004 |
| Registro, email Apple sin OTP, continuación y cancelación | RP-APL-005..012 |
| Sesión, minimización, contraseña y cambio de email | RP-APL-013..016 |
| Métodos de acceso a una misma cuenta | RP-APL-017..019 |
| Cuenta Apple a email y contraseña | RP-APL-015..016; RP-APL-026 |
| Cuenta con email y contraseña a Apple | RP-APL-027..031 |
| Dos cuentas MDL ya creadas | DEC-01 |
| Revocación, notificaciones y disponibilidad | RP-APL-020..025 |
| Baja e intentos de revocación | RP-BAJ-016; RP-AUT-005..007 |

### Recorridos definidos y ejecución pendiente

Los dos recorridos aprobados se definen únicamente en [Cuenta](../../../rules/ACCOUNT.md#metodos-acceso-apple). La ejecución debe cubrir ambos sentidos, su acceso desde los ajustes cuando corresponda, la verificación de cambios de email y la conservación de la misma cuenta y Vault.

Antes de implementar esta ampliación deben reconciliarse los contratos, fases, escenarios y pruebas locales de API e iOS con esas reglas. No se presume cobertura por la existencia de `linkRequired` o `registrationRequired`. La resolución de dos cuentas distintas ya creadas permanece en [DEC-01](../../../rules/ACCOUNT.md#dec-01); no bloquea definir el recorrido normal de una única cuenta.

## Escenarios de aceptación

Los escenarios de alta y acceso referencian el catálogo. GWT-APPLE-05/06 cubren parte del recorrido de vinculación; no sustituyen la cobertura pendiente de ambos casos completos de RP-APL-015..019 y RP-APL-026..031. Los recorridos `linkRequired`/`registrationRequired` deben contrastarse con esas reglas antes de aceptar su implementación.

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

**Dado** un intento Apple válido sin los requisitos de nacimiento/mayoría de edad de `RP-CUE-001`/`RP-APL-005` o aceptaciones válidas
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
- [ ] La matriz E2E anti-duplicación anterior demuestra el mismo `User` y Vault para email coincidente y Private Relay/email distinto.
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
| iOS | `linkRequired`, alta Apple y baja con Vault activo validados parcialmente; `IOS-APPLE-03R` añade el vínculo manual desde `registrationRequired` con build y pruebas dirigidas verdes | ejecutar primero el vínculo anti-duplicación en dispositivo y luego revocación/validaciones previas a beta |
| Transversal | código principal implementado; beta readiness pendiente | Private Relay/notificaciones y revocación reales, operación de la outbox y disclosure público |

## Planes locales

- [API](../../../../../api/docs/plans/active/sign-in-with-apple/README.md)
- [iOS](../../../../../ios/docs/plans/active/sign-in-with-apple/README.md)

## Referencias

- [Cuenta de usuario](../../../rules/ACCOUNT.md)
- [`ARCHITECTURE.md` §2-4](../../../ARCHITECTURE.md#2-propiedad-de-datos)
- [Apple — Authenticating users with Sign in with Apple](https://developer.apple.com/documentation/signinwithapple/authenticating-users-with-sign-in-with-apple)
- [Apple — AuthenticationServices / credential state](https://developer.apple.com/documentation/authenticationservices/asauthorizationappleidprovider)
- [Apple — Token validation](https://developer.apple.com/documentation/signinwithapplerestapi/generate-and-validate-tokens)
- [Apple — Token revocation](https://developer.apple.com/documentation/signinwithapplerestapi/revoke-tokens)
- [Apple — Account deletion and revocation](https://developer.apple.com/documentation/technotes/tn3194-handling-account-deletions-and-revoking-tokens-for-sign-in-with-apple)
- [Apple — Sign in with Apple HIG](https://developer.apple.com/design/human-interface-guidelines/sign-in-with-apple/)

## Decisión abierta

1. **Dos cuentas MDL distintas ya creadas:** resolver [DEC-01 de Cuenta](../../../rules/ACCOUNT.md#dec-01) antes de implementar un tratamiento de esa colisión. Los recorridos normales de una misma cuenta ya están definidos y su implementación completa permanece pendiente.
2. **Revisión Legal previa al rollout:** el inventario y la retención técnica de Apple ya están fijados; faltan aprobación de privacy disclosures, ficha App Store Privacy, proveedor/transferencias y texto público. No bloquea las fases técnicas sin activación, pero sí cualquier rollout público.
