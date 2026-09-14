# Verificación de email

> **Estado:** recuperación de bootstrap iOS implementada y validada físicamente; limpieza de Home pendiente
> **Proyectos:** API e iOS
> **Última actualización:** 2026-09-07

## Objetivo

Comprobar que el usuario controla el email antes de habilitar el uso funcional de la cuenta y la creación del Vault.

## Alcance

- Envío automático de un código al registrarse.
- Ingreso y reenvío del código desde iOS.
- Estado de verificación y OTP dentro del gate obligatorio; Home y las superficies funcionales no muestran recordatorios ni CTA de verificación.
- Gate para configurar Vault, cambiar email y eliminar cuenta; cualquier cambio de contraseña autenticado futuro deberá usar el mismo gate. El reset OTP anónimo vigente no forma parte de esta regla.
- Verificación del nuevo email antes de activar un cambio de dirección.
- Cuenta provisional con plazo fijo de siete días y baja automática si no verifica.

Fuera de alcance:

- Sincronización remota de fotos.
- Cambios al flujo de recuperación de contraseña.
- Webhooks de bounce del proveedor de email.
- Verificación manual sin procedimiento auditado.

## Comportamiento esperado

### Registro y verificación

- El registro crea una cuenta provisional y una sesión limitada al flujo de verificación; no habilita Home, diario, Vault ni acciones funcionales.
- API envía un código numérico de seis dígitos con vigencia de 15 minutos.
- Excepción coordinada: una cuenta nueva creada mediante [Sign in with Apple](../sign-in-with-apple/README.md) nace verificada cuando API valida el claim `email_verified`; no genera OTP. Vincular Apple a una cuenta existente no cambia su email ni su verificación.
- Existen como máximo cinco intentos antes de un bloqueo de 15 minutos.
- El reenvío aplica cooldown progresivo y sustituye el código anterior.
- La respuesta no permite descubrir si un email pertenece a una cuenta.
- Los códigos no se conservan ni registran en claro.
- El plazo de verificación es de siete días desde el alta. Reenviar códigos, iniciar sesión o crear un OTP nuevo no lo reinician ni lo prolongan.
- Si vence el plazo sin verificar, se elimina íntegramente la cuenta provisional y el email queda libre para registrarse de nuevo. No se conserva como lead ni destinatario comercial.

### Acceso según estado

- Hasta verificar: la sesión sólo permite verificar/reenviar/consultar el estado de email y cerrar sesión. iOS muestra una pantalla de verificación obligatoria; no monta Home, Settings, Vault ni un banner alternativo. API rechaza las operaciones funcionales como defensa final.
- Tras verificar: se habilita el acceso normal inmediatamente y, para una cuenta nueva, se presenta entonces el onboarding de Vault.
- Al vencer los siete días: la cuenta provisional se elimina; no existe suspensión prolongada ni retención automática como lead.

### Funciones sensibles

Requieren email verificado —y, en la práctica, ninguna operación funcional está disponible antes de verificar:

- configurar el Vault;
- cambiar contraseña desde una sesión autenticada, si esa función se incorpora; el reset anónimo actual queda excluido;
- cambiar email;
- eliminar la cuenta.

iOS explica el bloqueo y lleva directamente al flujo de verificación. API aplica la misma regla como defensa final.

### Cambio de email

- Solo un usuario con email actual verificado puede iniciar el cambio.
- El código se envía a la nueva dirección.
- La dirección anterior permanece activa hasta verificar la nueva.
- Tras confirmar, el nuevo email queda activo y verificado en una única operación coherente.

### Cuenta existente de beta

La única cuenta existente de beta se marca operativamente como verificada antes de activar el régimen de cuentas provisionales. No existe una migración masiva ni una campaña de retención de usuarios históricos en este alcance.

## Interacción entre proyectos

| Proyecto | Responsabilidad funcional |
|---|---|
| API | Generar, enviar y validar códigos; persistir y hacer cumplir el plazo de siete días; permitir sólo el conjunto mínimo de rutas durante la sesión provisional; ejecutar la baja automática |
| iOS | Mostrar la pantalla obligatoria, capturar el código, gestionar reenvío/cooldown y no iniciar Vault ni montar superficies funcionales antes de verificar |

## Orden general

1. API e iOS implementan OTP, envío, cliente, UI y gates básicos.
2. API introduce el régimen de cuenta provisional: plazo, allowlist de rutas y baja automática; iOS lo aplica como gate global de sesión.
3. Se ejecutan validación end-to-end, configuración del mailer, scheduler de vencimiento y prueba de baja automática.
4. Recuperación implementada: iOS usa `GET /auth/email/status` como validación de sesión antes de perfil/Vault, también en restauración en frío. API conserva su allowlist restrictivo; queda validación física contra ella.
5. Limpieza pendiente: eliminar el banner y guards redundantes de Home; una cuenta no verificada nunca alcanza esa superficie.

## Criterios de aceptación

- [x] El registro envía un código sin bloquear la creación de cuenta si el mailer falla temporalmente.
- [x] Código, expiración, intentos, lockout y reenvío se comportan según las reglas.
- [x] Una cuenta provisional no alcanza Home, diario ni Vault antes de verificar; API aplica el mismo límite.
- [x] Verificar desbloquea la app sin una nueva autenticación innecesaria, con evidencia automatizada y física.
- [x] El cambio de email no abandona la dirección anterior antes de verificar la nueva.
- [x] No hay códigos en logs, respuestas o almacenamiento local persistente.
- [x] Registro/login/restauración iOS revalidados físicamente contra el régimen provisional activo; mailer, API y scheduler activos.
- [ ] Confirmar la primera ejecución nocturna sin errores y, antes del cierre documental, la baja real de una cuenta provisional vencida sin dejar tokens, Vault ni artefactos locales activos.

## Estado resumido

| Proyecto | Estado | Próximo resultado |
|---|---|---|
| API | OTP, envío, allowlist global, vencimiento y baja automática desplegados; flag de régimen activo | Sin cambio: el allowlist no debe habilitar perfil ni Vault antes de verificar. |
| iOS | Bootstrap provisional aceptado y validado: alta/login/restauración consultan estado antes de perfil/Vault; `validateSession()` basado en perfil fue retirado | Retirar el banner/CTA redundante de Home. |
| Transversal | Cobertura automatizada y validación física confirman OTP, reanudación sin relogin y restauración pendiente | Mantener el gate como única entrada para una cuenta pendiente. |
| iOS — limpieza Home | El banner/CTA de verificación y guards asociados son remanentes de un flujo anterior al gate global | Retirarlos: Home sólo existe para cuentas verificadas. |

## Planes locales

- API: `api/docs/plans/active/email-verification/README.md`
- iOS: `ios/docs/plans/active/email-verification/README.md`

## Referencias estables

- Privacidad: `product/docs/legal/README.md`
- Recuperación existente (mismo patrón OTP): `api/app/Contexts/Identity/Infrastructure/Service/PasswordResetService.php`
- Vault estable: [`product/docs/VAULT.md`](../../../VAULT.md)
