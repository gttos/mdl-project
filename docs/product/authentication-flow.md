# Flujo de Autenticación y Sesión — MDL

> **Ámbito:** Producto completo (iOS + Backend).  
> **Propósito:** Describir el comportamiento compartido de autenticación, sesión, recuperación de contraseña y App Lock, sin entrar en detalles de implementación de framework.

## 1. Principios de producto

- **Impermeable:** La parte autenticada de la app nunca es accesible sin sesión válida.
- **Global:** La expiración de sesión es un evento global gestionado por el networking layer, no por pantallas individuales.
- **Separado:** La seguridad del dispositivo (App Lock / biometría) es independiente de la seguridad de red (token de sesión). Uno bloquea la UI; el otro invalida la sesión.
- **Recuperable:** El usuario puede recuperar su contraseña mediante un flujo OTP de 3 pasos sin revelar si su email está registrado.

## 2. Ciclo de autenticación

### 2.1 Registro

1. Usuario ingresa email, contraseña, fecha de nacimiento y acepta términos y política de privacidad activos.
2. Backend valida unicidad de email, fuerza de contraseña, edad mínima (≥ 18 años) y versiones de términos/privacidad.
3. Backend crea usuario con email normalizado, contraseña hasheada (bcrypt) y asigna plan básico.
4. Backend registra aceptación legal con IP y user agent.
5. Backend emite token Sanctum con fecha de expiración (30 días).
6. App iOS almacena el token en Keychain (no en UserDefaults).

### 2.2 Inicio de sesión

1. Usuario ingresa email y contraseña.
2. Backend verifica credenciales, usuario activo, y emite token Sanctum.
3. Backend registra evento de autenticación (`AuthEvent`) con outcome.
4. App iOS almacena token en Keychain y resetea flags de inactividad.

### 2.3 Cierre de sesión

1. Usuario cierra sesión desde la app.
2. Backend revoca el token Sanctum actual.
3. App iOS elimina el token del Keychain.
4. App iOS redirige al flujo de onboarding/login.

### 2.4 Expiración de sesión

- Los tokens Sanctum expiran a los 30 días.
- Si el backend responde `401` o `403`, el networking layer de iOS emite un evento global de sesión inválida.
- El `AuthViewModel` de iOS escucha este evento, purga el token del Keychain y redirige al login.
- **Regla:** ninguna pantalla individual maneja el `401` localmente.

## 3. Recuperación de contraseña (OTP)

### 3.1 Flujo de 3 pasos

1. **Solicitud:** `POST /api/auth/password/forgot` con email. El backend responde igual si el email existe o no (anti-enumeración). Si existe, genera un OTP de 6 dígitos, lo hashea y envía un email con el código.
2. **Verificación:** `POST /api/auth/password/verifycode` con email y código. El backend verifica el hash y marca el OTP como verificado si es correcto.
3. **Reset:** `POST /api/auth/password/reset` con email, código verificado y nueva contraseña. El backend actualiza la contraseña y elimina el OTP.

### 3.2 Reglas de throttling y seguridad

- Cooldown progresivo entre reenvíos: 60s, 300s, 3600s según cantidad de intentos.
- Bloqueo tras 5 intentos fallidos de verificación.
- Rate limit por IP en rutas públicas de autenticación.
- Los OTP expiran tras un período definido y se purgan automáticamente a las 24 horas post-expiración.

## 4. App Lock (bloqueo local)

- El usuario puede activar un bloqueo local con PIN o biometría (Face ID / Touch ID).
- Al volver a primer plano, la app muestra un overlay de bloqueo sin destruir el estado de navegación subyacente.
- El App Lock **no** invalida el token de sesión del backend. Es una capa de seguridad del dispositivo.
- La VMK del Vault también se protege con biometría en el Keychain (`kSecAccessControlBiometryCurrentSet`), pero eso es independiente del App Lock general.

## 5. Contrato entre cliente y backend

| Acción | Endpoint (backend) | Payload mínimo | Respuesta esperada |
|---|---|---|---|
| Registro | `POST /api/register` | email, password, birth_date, accept_terms, terms_version, accept_privacy, privacy_version | Token + datos de usuario |
| Login | `POST /api/login` | email, password | Token + datos de usuario |
| Logout | `POST /api/logout` | Bearer token | Confirmación |
| Forgot password | `POST /api/auth/password/forgot` | email | Mensaje genérico |
| Verify OTP | `POST /api/auth/password/verifycode` | email, code | Confirmación |
| Reset password | `POST /api/auth/password/reset` | email, code, password | Confirmación |

## 6. Implementaciones específicas por proyecto

- **iOS:** consultar `ios/docs/domains/features/auth-session.md` para detalles de `HTTPClient`, `SceneDelegate`, `AuthViewModel`, `KeychainHelper` y manejo de `401` global.
- **Backend:** consultar `api/docs/security/security.spec.md` y `api/docs/features/auth-recovery-flow.spec.md` para detalles de Sanctum, `PasswordResetService`, `AuthLogger`, policies y rate limiting.

## 7. Cambios transversales pendientes

- El Vault introduce una **Recovery Password** independiente de la contraseña de login. Esto cambia el modelo mental del usuario: ahora tiene dos contraseñas (login y Vault). La documentación de UX y legal debe reflejar esta separación.
- Los tokens de sesión existentes no tienen relación con la VMK del Vault. La autenticación de la app y el desbloqueo del Vault son procesos separados.
