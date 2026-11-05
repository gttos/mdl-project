# Acceso Google en Android

<a id="objetivo"></a><a id="alcance"></a><a id="decisiones-y-contrato-compartido"></a><a id="interacción-entre-proyectos"></a><a id="orden-y-fases-transversales"></a><a id="estado-técnico-vigente"></a><a id="gates-de-distribución-y-legal"></a><a id="próximo-resultado-permitido"></a><a id="planes-locales"></a><a id="referencias"></a>

> **Estado:** AND-GOOGLE-01/02 y API-GOOGLE-01/02 aceptadas con evidencia automatizada. API/AND-GOOGLE-03 pendientes; validación con credenciales reales y distribución pendientes.

## Alcance

Crear, abrir y vincular el acceso Google a la misma cuenta, sesión y diario desde Android, conservando la acreditación explícita y la separación entre cuentas.

- [Definición futura aprobada, RP-GGL-001..021](../../../FUNCIONES_FUTURAS_PARA_JIRA.md#google).
- [Condiciones pendientes de activación](../../../DECISIONES_PARA_JIRA.md#activacion-google).

## Escenarios de aceptación

<a id="gwt-google-01--separación-por-plataforma"></a>
### GWT-GOOGLE-01 — Separación por plataforma

**Dado** el flujo no autenticado de Android
**Cuando** se habilita la función en una distribución Android futura
**Entonces** ofrece email/contraseña y Google, no Apple; iOS ofrece email/contraseña y Apple, no Google.

<a id="gwt-google-02--login-de-identidad-vinculada"></a>
### GWT-GOOGLE-02 — Login de identidad vinculada

**Dado** una identidad Google válida vinculada a un usuario activo
**Cuando** API valida token, audiencia, vigencia, nonce y replay
**Entonces** emite una sesión MDL para esa misma cuenta y Android abre su mismo Vault.

<a id="gwt-google-03--alta-nueva"></a>
### GWT-GOOGLE-03 — Alta nueva

**Dado** una identidad Google válida no vinculada con email acreditado
**Cuando** la persona declara una fecha que acredita mayoría de edad y acepta las versiones legales vigentes
**Entonces** se crean atómicamente la cuenta, la identidad Google y sus aceptaciones y se inicia sesión sin contraseña ni OTP MDL.

<a id="gwt-google-04--sin-merge-automático"></a>
### GWT-GOOGLE-04 — Sin merge automático

**Dado** que el email Google coincide con una cuenta MDL existente
**Cuando** todavía no se acreditó esa cuenta
**Entonces** no se vincula ni se emite sesión y se ofrece el recorrido explícito de vinculación.

<a id="gwt-google-05--vinculación-explícita"></a>
### GWT-GOOGLE-05 — Vinculación explícita

**Dado** una cuenta MDL existente y una identidad Google todavía libre
**Cuando** la persona acredita ambos accesos y confirma la vinculación
**Entonces** Google se añade a la misma cuenta sin cambiar email, identidad, Vault ni diario.

<a id="gwt-google-06--cancelación-y-fallo"></a>
### GWT-GOOGLE-06 — Cancelación y fallo

**Dado** el selector de Google abierto
**Cuando** la persona cancela o la credencial falla
**Entonces** Android permanece no autenticado, conserva el formulario y diferencia cancelación neutral de fallo reintentable sin crear estado durable.

<a id="gwt-google-07--credencial-inválida-o-replay"></a>
### GWT-GOOGLE-07 — Credencial inválida o replay

**Dado** un token con firma, emisor, audiencia, vigencia o nonce inválidos, o una continuación ya consumida
**Cuando** llega a API
**Entonces** se rechaza sin crear, vincular o emitir sesión y sin registrar credenciales.

<a id="gwt-google-08--aislamiento-de-cuenta"></a>
### GWT-GOOGLE-08 — Aislamiento de cuenta

**Dado** que el dispositivo cambia de una cuenta A a otra B mediante Google
**Cuando** Android activa la nueva sesión
**Entonces** publica autenticación sólo después de activar el Vault de B y no mezcla claves, entradas, pendientes ni fotos de A.

<a id="gwt-google-09--acceso-entre-plataformas"></a>
### GWT-GOOGLE-09 — Acceso entre plataformas

**Dado** una cuenta creada con Google
**Cuando** añade y verifica su primera contraseña MDL
**Entonces** puede acceder por email/contraseña desde iOS a la misma cuenta y Vault sin retirar Google.

<a id="gwt-google-10--revocación-o-indisponibilidad"></a>
### GWT-GOOGLE-10 — Revocación o indisponibilidad

**Dado** que Google revoca la autorización o no está disponible
**Cuando** MDL recibe una señal verificable o falla la autorización
**Entonces** impide el acceso Google y revoca las sesiones afectadas cuando corresponda, pero no borra el diario; una indisponibilidad permite reintentar y nunca simula una sesión.

## Orden y estado

| Fase | Resultado | Estado |
|---|---|---|
| AND-GOOGLE-01 | Autorización nativa aislada. | Aceptada con evidencia automatizada. |
| API-GOOGLE-01 | Verificación y continuación de uso único. | Aceptada. |
| API-GOOGLE-02 | Login, alta, vínculo explícito y primera contraseña. | Aceptada con evidencia automatizada. |
| AND-GOOGLE-02 | Acceso público y vínculo desde sesión existente, sin adoptar otra cuenta. | Aceptada con evidencia automatizada; proveedor real pendiente. |
| API/AND-GOOGLE-03 | Revocación, baja y limpieza de credenciales. | Pendiente; siguiente corte API-GOOGLE-03. Android espera su contrato. |
| GOOGLE-04 | Configuración, revisión humana/legal, validación física e integración. | Pendiente antes de distribución. |

## Planes locales y salida

- [API](../../../../../api/docs/plans/active/sign-in-with-google/README.md).
- [Android](../../../../../android/docs/plans/active/sign-in-with-google/README.md).

El acceso público permanece deshabilitado hasta completar sus condiciones. Cada fase mantiene su evidencia y límites en el proyecto responsable. El cierre conserva la validación conjunta y reconciliación documental.
