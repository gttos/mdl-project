# Información de la cuenta de usuario

**Estado:** definición de producto actualizada; implementación pendiente. El registro comprueba la mayoría de edad y conserva la fecha completa declarada junto con la fecha de alta, conforme a `RP-CUE-001`. Nombre, teléfono y edad exacta quedan fuera de la cuenta.

## Objetivo y alcance

Aplicar las decisiones de [Cuenta de usuario](../../../rules/ACCOUNT.md): `RP-CUE-001`, `RP-APL-005`, `RP-PER-002`, `RP-PER-007`, `RP-PER-008` y `RP-VLT-005`. Incluye cuentas nuevas y existentes, alta por email y Apple, perfil, almacenamiento, búsquedas de identidad, comunicaciones y retirada de datos anteriores.

Las reglas pertenecen a ACCOUNT; este plan organiza su implementación. No cambia el contenido privado del Crush, sus edades/cumpleaños, sus reglas de género ni la criptografía del Vault. Tampoco activa nuevos proveedores, resuelve la fusión de cuentas ni crea una consola de soporte.

## Contrato transversal

- El email se cifra a nivel de campo en el servidor. El backend necesita recuperarlo para las funciones autorizadas de la cuenta; no es cifrado de extremo a extremo ni anonimización. Las claves no residen en la base que contiene los datos.
- La búsqueda exacta y la unicidad usan una representación protegida del email normalizado, sin mantener una copia legible para consultar. Se preserva `BR-3.03`; no se introducen equivalencias de puntos o alias específicas de Gmail.
- El transporte sigue protegido por HTTPS. La app puede enviar el email para autenticarse y mostrar a la persona su dirección; esta feature no convierte el email en un secreto inaccesible al backend ni exige desbloquear el Vault para recuperar acceso.
- El registro y la corrección del nacimiento reciben `birth_date` como fecha civil completa para comprobar los 18 años cumplidos. API conserva esa fecha declarada y la devuelve como `birthDate`; `created_at` conserva cuándo se realizó el alta. No se persiste además una edad exacta ni se presenta la declaración como una verificación de identidad.
- Se retiran nombre y teléfono del perfil, sus contratos, persistencia y copias operativas. Se conservan `gender_id` y `country_id` y sus catálogos actuales. No se cambia su obligatoriedad por iniciativa técnica.
- La conversión de cuentas existentes mantiene ID, fecha de nacimiento declarada, fecha de alta, verificación, credenciales, vínculos Apple, diario y namespace del Vault. Los datos ausentes o incoherentes no se completan con valores inventados.

## Escenarios de aceptación

| ID | Dado / Cuando / Entonces | Reglas |
|---|---|---|
| GWT-UI-01 | Dada una cuenta nueva o existente, cuando se guarda o migra su email, entonces la base no conserva la dirección legible y el acceso con el mismo email sigue funcionando sin distinguir mayúsculas. | RP-VLT-005; BR-3.03 |
| GWT-UI-02 | Dado un email cifrado, cuando se solicita recuperación, verificación o un aviso de inactividad, entonces el envío llega al destinatario previsto y se mantienen los bloqueos, plazos y respuestas públicas vigentes. | RP-VLT-005; BR-3.06; RP-VER-002; RP-OTP-001 |
| GWT-UI-03 | Dado un cambio de email, cuando aún falta verificar la dirección nueva, entonces sigue activa la anterior y ambas direcciones almacenadas están protegidas; al confirmar, dirección y búsqueda cambian juntas. | RP-VLT-005; RP-MAIL-002..003 |
| GWT-UI-04 | Dada una cuenta Apple con correo personal o intermediario, cuando entra o utiliza un método de acceso ya vinculado, entonces encuentra la misma cuenta sin crear otra ni modificar su verificación. | RP-APL-006; RP-APL-017..018; RP-VLT-005 |
| GWT-UI-05 | Dado el perfil de una cuenta, cuando se consulta, edita o migra, entonces conserva `birth_date`, género y país, no conserva una edad exacta y no conserva nombre ni teléfono. | RP-CUE-001; RP-PER-002; RP-PER-007..008 |
| GWT-UI-06 | Dada una persona que todavía tiene 17 años, aunque cumpla 18 durante este año, cuando intenta registrarse por cualquier canal, entonces se rechaza el alta según la fecha completa declarada; cumplir 18 hoy sí permite registrarse y la fecha declarada queda asociada a la fecha del alta. | RP-BASE-001; RP-CUE-001; RP-APL-005 |
| GWT-UI-07 | Dada una búsqueda operativa autorizada por email completo, cuando se localiza la cuenta, entonces se obtiene la misma identidad mediante la búsqueda protegida, sin volcar emails a diagnósticos ni acceder al contenido del Vault. | RP-VLT-005; reglas de [Privacidad](../../../rules/PRIVACY.md) |
| GWT-UI-08 | Dada una migración interrumpida o una restauración, cuando se reanuda el servicio, entonces no se pierden cuentas ni vínculos, no reaparecen datos retirados en el uso ordinario y no se habilita un fallback a email legible. | RP-PER-008; RP-VLT-005; [Ciclo de vida](../../../rules/DATA_LIFECYCLE.md) |

## Orden y responsabilidades

| Fase | Resultado | Estado |
|---|---|---|
| UI-00 — contrato y preparación | Cerrar mapa de consumidores, custodia/rotación de claves, tratamiento de datos existentes y orden de corte. | Pendiente; primera fase ejecutable de preparación, sin migrar datos |
| API-UI-01 — email cifrado y búsquedas | Cifrado, consulta exacta, unicidad y adaptación de todos los flujos de identidad/comunicación, con pruebas. | Pendiente |
| API-UI-02 — perfil mínimo | Contrato de fecha de nacimiento declarada, retirada de edad/nombre/teléfono y migración preparada y ensayada con datos sintéticos. | Pendiente; requiere contrato API |
| IOS-UI-01 — cliente iOS | Registro, Apple, perfil, payloads y almacenamiento local adaptados al contrato final. | Pendiente; requiere contrato API |
| AND-UI-01 — cliente Android | Registro y perfil con el mismo contrato; no activa Apple en Android. | Pendiente; requiere contrato API |
| UI-03 — corte y cierre | Migración autorizada, builds compatibles, validación integrada, conciliación documental y cierre coordinado. | Pendiente; operación sobre datos reales requiere autorización específica |

Una fase local por sesión. No se cambia el contrato de nacimiento en un entorno utilizado por clientes antiguos hasta disponer de builds compatibles y un corte coordinado. UI-00 fija el orden de despliegue, manejo de escrituras concurrentes/colas y recuperación ante fallos; no queda una compatibilidad histórica permanente en la versión pública.

API prepara persistencia y contrato; iOS y Android adaptan sus consumidores. Product acepta las fases y actualiza primero el estado maestro. Los planes locales están en [API](../../../../../api/docs/plans/active/user-information/README.md), [iOS](../../../../../ios/docs/plans/active/user-information/README.md) y [Android](../../../../../android/docs/plans/active/user-information/README.md).

## Relación con trabajo activo

- [Email verification](../email-verification/README.md): se preservan OTP, activación del email nuevo y baja provisional; aquí se adapta su almacenamiento y consulta.
- [Sign in with Apple](../sign-in-with-apple/README.md): aquí se adapta email/nacimiento; los recorridos de vinculación pendientes siguen en ese plan. Su evidencia anterior no certifica el nuevo contrato.
- [Security remediation](../security-remediation/README.md): conserva la remediación de credenciales, sesiones y observabilidad general. Antes de editar código compartido se reconcilia el estado de ambas iniciativas; aquí se cubren las fugas y consultas afectadas por el cifrado, sin duplicar correctivos.
- [Android parity](../android-parity/README.md): esta feature asume el cambio de cuenta; la aceptación anterior de autenticación/perfil no demuestra este nuevo contrato.
- [Business rule integrity](../business-rule-integrity/README.md): continúa la revisión del catálogo y la auditoría posterior. Este plan no declara cumplimiento global.

## Privacidad y operación

El inventario de [Legal](../../../legal/02_MDL_Inventario_Maestro_Datos.md) describe todavía fecha completa y teléfono; es evidencia del modelo anterior, no aprobación del nuevo tratamiento. La revisión jurídica de minimización, comprobación de edad, información pública y backups queda pendiente. No se modifica Legal en esta fase.

Antes del corte debe quedar definido cómo retirar nombre, teléfono y día/mes de las copias operativas, sin prometer su desaparición instantánea de backups o proveedores. La política de [conservación y restauración](../../../legal/04_MDL_Ciclo_Vida_Retencion_Derechos.md) se respeta y se valida por separado; no se inventan plazos. Las claves y el índice de búsqueda siguen siendo material sensible y no convierten el email en anónimo.

## Criterio de cierre

Todos los GWT aplicables tienen evidencia API y de los clientes, incluidas cuentas existentes, Private Relay y envíos reales autorizados. La retirada efectiva de datos y la protección de colas, cachés, logs y backups están comprobadas en su alcance correspondiente; las validaciones manuales y jurídicas exigidas están resueltas. Se integra el estado técnico estable en los documentos canónicos y se eliminan juntos este plan y los tres locales. Hoy no hay código, tests ni migraciones ejecutados por esta iniciativa.
