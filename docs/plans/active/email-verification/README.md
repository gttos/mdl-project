# Verificación de email

> **Estado:** pendiente de implementación
> **Proyectos:** API e iOS
> **Última actualización:** 2026-07-18

## Objetivo

Comprobar que el usuario controla el email de su cuenta y usar ese estado para comunicaciones operativas y para proteger acciones sensibles, sin bloquear el uso básico durante el periodo inicial de verificación.

## Alcance

- Envío automático de un código al registrarse.
- Ingreso y reenvío del código desde iOS.
- Estado visible en Home y Settings.
- Gate para configurar Vault, cambiar contraseña, cambiar email y eliminar cuenta.
- Verificación del nuevo email antes de activar un cambio de dirección.
- Tratamiento de usuarios existentes y despliegue compatible.

Fuera de alcance:

- Sincronización remota de fotos.
- Cambios al flujo de recuperación de contraseña.
- Webhooks de bounce del proveedor de email.
- Verificación manual sin procedimiento auditado.

## Comportamiento esperado

### Registro y verificación

- El registro completa y la sesión puede iniciarse aunque el email todavía no esté verificado.
- API envía un código numérico de seis dígitos con vigencia de 15 minutos.
- Existen como máximo cinco intentos antes de un bloqueo de 15 minutos.
- El reenvío aplica cooldown progresivo y sustituye el código anterior.
- La respuesta no permite descubrir si un email pertenece a una cuenta.
- Los códigos no se conservan ni registran en claro.

### Acceso según estado

- Días 0 a 7: la app básica sigue disponible y Home muestra un recordatorio.
- Días 8 a 30: las funciones sensibles están bloqueadas y se muestra una advertencia.
- Desde el día 31: la cuenta queda suspendida, no eliminada; tras login solo se ofrece verificar el email.
- La verificación correcta desbloquea el acceso inmediatamente.

### Funciones sensibles

Requieren email verificado:

- configurar el Vault;
- cambiar contraseña;
- cambiar email;
- eliminar la cuenta.

iOS explica el bloqueo y lleva directamente al flujo de verificación. API aplica la misma regla como defensa final.

### Cambio de email

- Solo un usuario con email actual verificado puede iniciar el cambio.
- El código se envía a la nueva dirección.
- La dirección anterior permanece activa hasta verificar la nueva.
- Tras confirmar, el nuevo email queda activo y verificado en una única operación coherente.

### Usuarios existentes

Los usuarios sin `email_verified_at` reciben el mismo proceso de verificación. No se consideran verificados únicamente por haber usado anteriormente la recuperación de contraseña. El envío masivo debe ser gradual y controlado desde el plan local de API.

## Interacción entre proyectos

| Proyecto | Responsabilidad funcional |
|---|---|
| API | Generar, enviar y validar códigos; exponer estado; aplicar límites y gate; ejecutar rollout/backfill controlado |
| iOS | Mostrar estado, capturar el código, gestionar reenvío/cooldown y bloquear la navegación sensible con una explicación accionable |

## Orden general

1. API implementa persistencia, envío, validación y consulta de estado sin activar todavía el gate restrictivo.
2. iOS integra consulta, banner y flujo de verificación.
3. API activa protección de acciones sensibles cuando una versión iOS compatible está disponible.
4. iOS completa gates y cambio de email.
5. Se ejecutan validación end-to-end, configuración de mailer y rollout gradual de usuarios existentes.

## Criterios de aceptación

- [ ] El registro envía un código sin bloquear la creación de cuenta si el mailer falla temporalmente.
- [ ] Código, expiración, intentos, lockout y reenvío se comportan según las reglas.
- [ ] Home y Settings reflejan el estado real de API.
- [ ] Las funciones sensibles se bloquean en iOS y API cuando corresponde.
- [ ] Verificar desbloquea la app sin una nueva autenticación innecesaria.
- [ ] El cambio de email no abandona la dirección anterior antes de verificar la nueva.
- [ ] No hay códigos en logs, respuestas o almacenamiento local persistente.
- [ ] Una app anterior no queda bloqueada por el middleware antes del rollout compatible.
- [ ] El mailer real y el envío gradual de usuarios existentes se validan antes de activar suspensión.

## Estado resumido

| Proyecto | Estado | Próximo resultado |
|---|---|---|
| API | Pendiente | API-01 — persistencia de códigos |
| iOS | Pendiente | iOS-01 después del contrato API o contra mocks aprobados |
| Transversal | Pendiente | Activación restrictiva solo después de compatibilidad iOS |

## Planes locales

- API: `api/docs/plans/active/email-verification/README.md`
- iOS: `ios/docs/plans/active/email-verification/README.md`

## Referencias estables

- Privacidad: `product/docs/legal/README.md`
- Recuperación existente: `api/docs/features/auth-recovery-flow.spec.md`
- Vault: `product/docs/plans/active/vault-e2ee/README.md`
