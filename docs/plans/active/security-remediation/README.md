# Remediación de hallazgos de seguridad de API

> **Estado:** planteamiento aprobado; ejecución diferida hasta el cierre coordinado de `relationship-redesign`. Ninguna fase de implementación está autorizada todavía.
> **Proyectos:** Product y API. iOS es consumidor contractual a comprobar; sólo se añadirá un plan local iOS si la fase de reconsolidación demuestra que un cambio de transporte o UX es inevitable.
> **Origen técnico:** escaneo estándar de Codex Security, offline y de solo lectura, completado el 14 de septiembre de 2026.

## Objetivo

Cerrar los hallazgos validados del escaneo de `api/` mediante fronteras de seguridad coherentes y verificables, sin convertirlos en diez parches aislados. La iniciativa protege identidad y sesiones, evita enumeración en autenticación pública, limita el presupuesto de almacenamiento del Vault y endurece los límites de despliegue, proxy y observabilidad.

El escaneo se selló sobre la revisión `a8f4b02e76c0ace514f9e2f5c1ff677680091ff9` mientras `develop` avanzaba. Sus hallazgos con evidencia de fuente se revalidaron contra el checkout posterior, pero la cobertura global quedó clasificada como parcial. Por eso la primera fase futura vuelve a fijar el baseline después de `relationship-redesign`; este plan no presupone que una línea concreta siga igual entonces.

## Alcance

- Mutaciones de email y contraseña, cambio y recuperación de credenciales, revocación de tokens y efectos laterales del login.
- Indistinguibilidad pública de login y recuperación de contraseña, cooldowns y límites de intentos concurrentes.
- URLs firmadas y reservas de subida del Vault, límites por objeto/usuario, verificación y cleanup de objetos temporales o no válidos.
- Exclusión de secretos del contexto e imagen Docker, allowlist de observabilidad y confianza explícita en proxies.
- Validaciones diferidas detectadas por el escaneo: límites de payload Apple, capacidad real de `plan_id`, fallback de URLs del Vault, advisories de dependencias y controles efectivos de infraestructura.
- Pruebas locales e in-process y uso del stack aislado ya preparado por `security-hardening`.

Fuera de alcance:

- Ejecutar ahora código, tests, contenedores, migraciones, deploys, rotaciones de claves o cambios de AWS/Cloudflare/Apple.
- Probar producción, proveedores, IPs externas, cuentas o datos reales; realizar fuerza bruta, carga o denegación de servicio.
- Abrir o copiar `.env`, claves `.p8`, credenciales, tokens o cualquier otro secreto.
- Declarar cumplimiento, certificar la criptografía o sustituir un pentest independiente.
- Rediseñar relaciones, Vault E2EE o Sign in with Apple más allá de las fronteras concretas afectadas por un hallazgo.

## Relación con iniciativas activas

- [`relationship-redesign`](../relationship-redesign/README.md) es el gate de inicio solicitado por el usuario. No se inicia `SEC-REM-00` hasta que su cierre coordinado haya reconciliado Product/API/iOS y el usuario autorice expresamente esta iniciativa.
- [`security-hardening`](../security-hardening/README.md) ya aporta rate limits, coalescencia de alertas y un stack Docker local aislado. Esta iniciativa reutiliza esas capacidades; no vuelve a construir el entorno ni duplica sus fases `05C/05D`.
- [`email-verification`](../email-verification/README.md) conserva la única activación válida de un email nuevo: OTP sobre la dirección destino y cambio atómico. El perfil genérico no puede saltarse ese contrato.
- [`sign-in-with-apple`](../sign-in-with-apple/README.md) conserva sus claims, revocación y secretos en runtime. Esta iniciativa sólo cierra la posibilidad de que un artefacto `.p8` entre al contexto o a una capa de imagen.
- [`vault-e2ee`](../vault-e2ee/README.md) sigue siendo autoridad sobre cifrado, bindings y lifecycle. Limitar bytes, reservas y cleanup no autoriza al backend a leer plaintext ni VMK.

## Baseline de riesgo a reconfirmar

| Frontera | Hallazgos de entrada | Resultado estructural buscado |
|---|---|---|
| Identidad y sesión | Un bearer puede cambiar email/contraseña desde el patch de perfil; los cambios de contraseña no revocan tokens; un login fallido puede cancelar la baja por inactividad | Credenciales fuera del perfil genérico, prueba reforzada para mutaciones sensibles, revocación explícita y efectos laterales sólo después de autenticar |
| Privacidad y abuso en auth pública | Login y cooldown de recuperación distinguen cuentas; las carreras de OTP quedaron pendientes de validación | Respuestas y coste comparables, límites independientes de la existencia de cuenta y consumo de OTP serializado |
| Capacidad de almacenamiento Vault | Longitud sin máximo, subida directa no ligada a bytes, endpoint temporal sin reserva y mismatch sin cleanup | Cada capacidad queda owner-bound, limitada por objeto y presupuesto, verificable y recuperable ante abandono |
| Despliegue y observabilidad | Un `.p8` puede entrar en la imagen; proxies wildcard; el fallback de errores registra input mediante denylist | Secretos fuera del contexto, proxies explícitos y logs construidos sólo desde campos técnicos permitidos |

Los diez hallazgos del escaneo se consideran entradas de planificación, no un estado eterno. `SEC-REM-00` los marca como vigentes, resueltos por trabajo concurrente o sustituidos por una causa nueva antes de tocar código.

## Reglas compartidas propuestas

Estas reglas son el objetivo de la iniciativa. Las decisiones abiertas al final deben resolverse antes de congelar la fase que dependa de ellas.

1. Un patch genérico de perfil no acepta ni ignora silenciosamente atributos de identidad, credencial o privilegio. Cambiar email usa el flujo OTP canónico; cambiar contraseña, si se ofrece autenticado, usa una operación dedicada con prueba reforzada.
2. La revocación tras cambio o recuperación de contraseña sigue `RP-OTP-007` de [Cuenta](../../../rules/ACCOUNT.md#recuperacion-acceso); credencial y revocación se confirman atómicamente.
3. Buscar una cuenta, verificar su actividad y cancelar un cierre por inactividad no muta nada hasta que las credenciales hayan sido verificadas correctamente.
4. Login, solicitud de recuperación y sus cooldowns no permiten distinguir cuenta inexistente, inactiva o con contraseña incorrecta mediante status, código público, estructura o trabajo criptográfico claramente omitido.
5. Los intentos y el consumo de OTP se deciden bajo una frontera transaccional capaz de representar concurrencia real; un límite no admite dos ganadores.
6. Una capacidad de subida del Vault queda ligada al propietario, a una reserva, a un máximo por objeto y a un presupuesto por cuenta. Expiración, abandono o mismatch no dejan almacenamiento huérfano indefinido.
7. `Forwarded`/`X-Forwarded-*` sólo se aceptan desde proxies conocidos. Una petición directa no puede elegir la IP o el esquema que usa rate limiting, HTTPS u observabilidad.
8. La observabilidad usa allowlist de metadata técnica. Nunca copia el payload completo ni depende de una lista de campos prohibidos que pueda quedar desactualizada.
9. Los secretos no forman parte del checkout, contexto Docker, capas de imagen, logs o artefactos de test. La aplicación los recibe sólo mediante el mecanismo operativo autorizado.

## Escenarios canónicos

### GWT-SEC-REM-01 — Perfil sin canal alternativo de credenciales

**Dado** un token válido y un patch de perfil con `email` o `password`
**Cuando** API procesa la petición
**Entonces** rechaza esos campos sin mutar perfil, verificación, credencial ni tokens, y el email sólo puede activarse mediante el flujo OTP canónico.

### GWT-SEC-REM-02 — Rotación de credencial y sesiones

**Dado** una contraseña cambiada o recuperada correctamente
**Cuando** la nueva credencial queda confirmada
**Entonces** todas las sesiones de esa cuenta, incluida la actual y las iniciadas mediante Apple, quedan revocadas en la misma operación; un bearer anterior ya no autentica y se exige un nuevo login, sin emitir automáticamente una sesión sustituta.

### GWT-SEC-REM-03 — Login sin efecto lateral previo

**Dado** una cuenta con cierre por inactividad pendiente
**Cuando** se intenta login con una contraseña incorrecta
**Entonces** la cuenta y su deadline permanecen intactos; sólo un login realmente autenticado cancela el cierre.

### GWT-SEC-REM-04 — Login no enumerable

**Dado** un email inexistente, una cuenta inactiva o una contraseña incorrecta
**Cuando** se intenta login
**Entonces** las tres respuestas públicas son indistinguibles y el detalle exacto sólo llega a observabilidad saneada.

### GWT-SEC-REM-05 — Recuperación no enumerable

**Dado** la misma secuencia de solicitudes para un email registrado y otro inexistente
**Cuando** se alcanza un cooldown o límite
**Entonces** status, envelope, código público y backoff no revelan cuál pertenece a una cuenta, sin enviar correo a la dirección inexistente.

### GWT-SEC-REM-06 — OTP bajo competencia

**Dado** dos verificaciones o consumos concurrentes del mismo OTP
**Cuando** la base los procesa con locks reales
**Entonces** como máximo uno gana y los intentos, bloqueo y consumo finales conservan un único estado válido.

### GWT-SEC-REM-07 — Subida Vault acotada

**Dado** un usuario autenticado que pide o reutiliza una capacidad de subida
**Cuando** declara, envía o verifica más bytes de los permitidos, supera su presupuesto o abandona la operación
**Entonces** la capacidad falla cerrada o expira, no puede reasignarse a otro propietario y el objeto no queda huérfano indefinidamente.

### GWT-SEC-REM-08 — Origen y proxy confiables

**Dado** una petición que llega fuera del proxy autorizado
**Cuando** controla cabeceras forwarded
**Entonces** no puede falsear la IP o el esquema efectivos ni eludir los límites asociados.

### GWT-SEC-REM-09 — Excepción sin fuga

**Dado** un payload con OTP, email destino, metadata o campos criptográficos del Vault
**Cuando** una excepción inesperada produce un `500`
**Entonces** el cliente recibe el error saneado y los logs contienen sólo contexto técnico explícitamente permitido.

### GWT-SEC-REM-10 — Imagen sin secretos locales

**Dado** un archivo local con extensión de clave que no pertenece al artefacto de aplicación
**Cuando** se construye la imagen
**Entonces** no entra al contexto ni aparece en ninguna capa; la aplicación obtiene el secreto únicamente en runtime.

## Orden de fases

| Fase | Resultado | Estado |
|---|---|---|
| `SEC-REM-00` — reconsolidación | Reindexado coordinado, escaneo estándar fresco, mapa de consumidores y decisiones de contrato | Pendiente; bloqueada por `relationship-redesign` y autorización de inicio |
| `API-SEC-REM-01` — secretos y observabilidad | Contexto/imagen sin claves locales y logs `500` por allowlist | Pendiente |
| `API-SEC-REM-02` — frontera de mutación de identidad | Perfil sin email/password y contrato dedicado de credencial resuelto | Pendiente; requiere decisión 1 |
| `API-SEC-REM-03` — lifecycle de sesión/login | Revocación tras contraseña y efectos laterales posteriores a autenticación | Pendiente de implementación; política de sesiones definida en RP-OTP-007 |
| `API-SEC-REM-04` — anti-enumeración y OTP | Login/reset indistinguibles y concurrencia real validada | Pendiente |
| `API-SEC-REM-05` — proxy y perímetro | Proxies explícitos, IP/esquema fiables y evidencia del origin | Pendiente; requiere decisión 4 y autorización operativa separada |
| `API-SEC-REM-06` — presupuesto Vault | Reservas owner-bound, límites por bytes, cleanup y lifecycle | Pendiente; requiere decisión 3 y posible ampliación a iOS |
| `API-SEC-REM-07` — validaciones diferidas | Apple, `plan_id`, fallback de storage, dependencias e infraestructura clasificados con evidencia | Pendiente |
| `SEC-REM-08` — regresión y cierre | Suites, scan de salida, validación local autorizada y reconciliación documental | Pendiente |

Cada fase se ejecuta en una sesión separada. Completar una fase no autoriza iniciar la siguiente.

## Interacción entre proyectos

| Proyecto | Responsabilidad |
|---|---|
| Product | Congelar invariantes, decisiones abiertas, orden, GWT y compatibilidad con Legal/Vault/email/Apple |
| API | Implementar las fronteras, tests de contrato/Feature/concurrencia y validaciones locales sin tráfico externo |
| iOS | Sólo validar que el contrato actual no se rompe; si cambia el método de upload o aparece una UX de cambio de contraseña, se añade alcance y plan local antes de implementar |
| Operaciones | Aportar, bajo autorización separada, rangos de proxy, política de origin, lifecycle/cuotas de objetos, inventario de imágenes y eventual rotación de credenciales |

## Criterios de aceptación

- Los diez hallazgos de entrada están cerrados con evidencia o reclasificados de forma explícita tras el scan fresco; ninguno desaparece por cambio de líneas o por asumir controles externos.
- Los GWT aplicables tienen tests trazables y los cambios de contrato incluyen todos sus consumidores, mocks y fixtures.
- Las invariantes de login, password, OTP y Vault que dependen de locks se prueban con un motor que represente concurrencia real.
- Ningún test, log, imagen o documento abre, copia o incorpora secretos reales.
- Las comprobaciones contra infraestructura o proveedores se ejecutan sólo con alcance, entorno y autorización específicos; los resultados no se infieren desde configuración local.
- El scan estándar final no conserva hallazgos reportables del alcance. La validación DAST/pentest, si se autoriza, reutiliza el gate previsto en `security-hardening`.
- El cierre integra el estado estable en Product y API y elimina este plan maestro y el local; no se conserva un informe paralelo.

## Decisiones de la iniciativa

Las decisiones todavía abiertas se resuelven antes de su fase dependiente; las ya definidas no acreditan implementación.

1. **Cambio autenticado de contraseña:** recomendar si esta iniciativa crea ya un endpoint dedicado con prueba de contraseña actual o si se limita a cerrar el canal inseguro del perfil y mantiene el reset OTP como único cambio disponible.
2. **Sesión posterior al cambio — definida:** aplicar `RP-OTP-007` de [Cuenta](../../../rules/ACCOUNT.md#recuperacion-acceso). Implementación pendiente en `API-SEC-REM-03`.
3. **Envelope de subida Vault:** fijar máximo por objeto, presupuesto por usuario y mecanismo compatible con el cliente — PUT con longitud/checksum firmados o policy POST con rango— después de medir snapshots válidos y el stack S3-compatible.
4. **Proxy confiable:** operaciones debe aportar los rangos/orígenes efectivos y confirmar si el origin es inaccesible fuera de Cloudflare. No se inventan CIDR ni se usa `*` como fallback.
5. **Compatibilidad iOS:** si la decisión 1 o 3 cambia payload, cabeceras o método HTTP, ampliar la iniciativa a iOS y crear su plan local antes de modificar API.

## Plan local

- [API](../../../../../api/docs/plans/active/security-remediation/README.md)
