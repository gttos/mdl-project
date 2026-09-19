# Integridad de reglas de negocio

> **Estado:** `ACCOUNT.md` y `CRUSH.md` están `FINALIZADO` por indicación expresa de Producto. `BRI-00B` **arrancó, limitado a ese subalcance** (Account y Crush); el resto del catálogo permanece en `EN REVISIÓN` y sin contraste. El contraste de este subalcance es de sólo lectura: no se implementó ningún fix, no se tocó código/tests/schema/proyecto Xcode y no se ejecutaron builds ni suites. La matriz completa vive en «Matriz BRI-00B — Account y Crush» más abajo. `BRI-00B` global sigue sin declararse completo; falta el resto de dominios y `BRI-00C` (revisión funcional de las discrepancias con el usuario) antes de abrir cualquier fase correctiva.

## 1. Objetivo

El catálogo de reglas tiene su índice en [PRODUCT.md](../../../PRODUCT.md) y sus definiciones por dominio en `docs/rules/`. Cada archivo está EN REVISIÓN hasta que el usuario lo dé por finalizado. Primero el usuario revisa qué debe hacer el producto; después esta iniciativa comprobará cada regla contra la implementación y propondrá correcciones. Este plan no es la fuente funcional ni impone ahora una matriz de cumplimiento.

Esta iniciativa no redefine el producto y no es una continuación de `relationship-redesign`. Su trabajo es:

1. identificar cada invariante estable o todavía en transición;
2. distinguir regla canónica, decisión legal pendiente, compatibilidad histórica y comportamiento accidental;
3. localizar la autoridad de API, las validaciones preventivas de iOS y los consumidores restantes;
4. demostrar cada regla con tests que fallen al revertirla;
5. retirar compatibilidad y bifurcaciones que no corresponden al lanzamiento greenfield;
6. ejecutar los fixes en fases pequeñas sólo después de que la matriz completa sea revisada y congelada.

No se implementa código durante `BRI-00`. Un test verde no convierte una implementación accidental en requisito.

## 2. Autoridad funcional

- Baseline de lanzamiento: `RP-BASE-006` en [Principios](../../../PRODUCT.md#principios).
- Baja y conservación seudonimizada: [Baja y ciclo de vida](../../../rules/DATA_LIFECYCLE.md); su decisión jurídica continúa abierta.
- Trayectos lógicos, creación, corrección y eliminación de hitos: [Relaciones](../../../rules/RELATIONSHIPS.md).
- Apple conserva el email acreditado por el proveedor sin OTP MDL. Los [dos recorridos para añadir un método de acceso](../../../rules/ACCOUNT.md#metodos-acceso-apple) están definidos y pendientes de implementación completa. Si el email o la identidad Apple ya pertenecen a otra cuenta MDL, la operación se rechaza conforme a `RP-MAIL-005` y `RP-APL-033`; no se fusionan ni transfieren cuentas.
- Los puntos sin definición suficiente se revisan en [Decisiones pendientes](../../../PRODUCT.md#decisiones-pendientes). No se convierten en incumplimientos de implementación de una regla inventada.

Los correctivos de `relationship-redesign` conservan su iniciativa y requieren alinear su contrato ejecutable con el catálogo antes de continuar; no se implementan en paralelo desde BRI.

## 3. Alcance y fuentes

### Incluye

- [`PRODUCT.md`](../../../PRODUCT.md), [`ARCHITECTURE.md`](../../../ARCHITECTURE.md), [`VAULT.md`](../../../VAULT.md), [`TIMELINE.md`](../../../TIMELINE.md) y [`STATISTICS.md`](../../../STATISTICS.md).
- Reglas todavía en transición de todos los planes maestros activos.
- Requisitos de [`legal/`](../../../legal/README.md) cuando condicionan privacidad, retención, borrado, acceso o presentación pública.
- Implementación y tests de API e iOS.
- Android y LLM sólo para demostrar que una función fuera de beta permanece inaccesible o cuando exista un consumidor real de una regla canónica.
- Rutas, jobs, comandos, flags, fixtures, seeds, snapshots y código muerto que puedan mantener compatibilidad histórica o eludir una invariante.

### No incluye todavía

- rediseño visual sin efecto funcional;
- activar Android, IA, IAP, personalidad, push completo o endpoints fuera de beta;
- despliegues, migraciones/backfills de datos reales, resets o borrados;
- modificar Legal o aprobar una cuestión jurídica;
- arreglar código durante la fase de inventario.

## 4. Método y estados

Cada regla queda registrada con:

1. fuente canónica exacta;
2. autoridad de enforcement en API;
3. preflight/presentación en iOS;
4. consumidores, mocks, fixtures y rutas alternativas;
5. tests positivos, negativos, idempotencia, rollback y concurrencia cuando correspondan;
6. validación manual u operativa no sustituible por tests;
7. estado y fase correctiva.

Estados permitidos:

| Estado | Significado |
|---|---|
| `CONFORME` | Código vigente y consumidores demostrados; tests adecuados ejecutados. |
| `PARCIAL` | Alguna frontera, consumidor o evidencia falta. |
| `INCUMPLIDA` | Existe un camino real que contradice la regla. |
| `NO DEMOSTRADA` | La regla existe, pero aún no hay auditoría fresca suficiente. |
| `CONTRADICCIÓN DOCUMENTAL` | Fuentes vigentes describen comportamientos distintos. |
| `DECISIÓN LEGAL` | La implementación depende de aprobación jurídica, no de una decisión del agente. |
| `FUERA DE BETA` | La capacidad debe permanecer inaccesible y sin consumidor productivo. |

Una regla no pasa a `CONFORME` por un comentario, un test que sólo replica la implementación, un reporte de otra sesión o una inspección sobre un índice desactualizado.

## 5. Hallazgos críticos ya identificados

| ID | Hallazgo | Estado |
|---|---|---|
| `BRI-DEL-001` | Product exige conservar la estructura desidentificada tras la baja, pero el flujo API documentado y observado para cuentas con Vault aplica `DeleteCrushUseCase` al grafo completo. El camino sin Vault conserva estructura. El resultado depende indebidamente de una bifurcación “con Vault/legacy”. | `INCUMPLIDA` |
| `BRI-DEL-002` | La purga Vault y la liberación de bindings están acopladas hoy al soft delete funcional. Preservar estructura requiere desacoplar destrucción criptográfica de borrado de Crush/Experience/período y demostrar que ningún binding/blob sobrevive al plazo. | `PARCIAL` |
| `BRI-DEL-003` | `AnonymizeCrushUseCase` elimina eventos de actividad, pero el flujo de baja observado anonimiza directamente Crushes en una rama. Debe demostrarse que siempre desaparecen los `ActivityEvent` y que no queda un feed personal recuperable. | `NO DEMOSTRADA` |
| `BRI-REL-001` | El documento `rules/RELATIONSHIPS.md` reúne períodos, hitos latest-only y trayectos lógicos; Timeline referencia ese contrato. Quedan por reconciliar el expediente Legal y la ejecución técnica anterior antes de aceptar fases dependientes; no se ha comprobado cumplimiento del código. | `CONTRADICCIÓN DOCUMENTAL` residual / contrato único definido |
| `BRI-REL-002` | La validación interactiva demostró que «Editar relación» podía agregar historia. `API-RR-04C` fue reportada, pero su aceptación coordinada y el correctivo iOS siguen fuera de esta auditoría hasta contar con grafo fresco y evidencia completa. | `PARCIAL` |
| `BRI-LEG-001` | Vault, durable write, relación, email y arquitectura todavía contienen rutas, modelos, fallbacks, fixtures o requisitos del “único usuario legacy”. La decisión greenfield los vuelve deuda a retirar, no compatibilidad a preservar. | `INCUMPLIDA` |
| `BRI-DOC-001` | Documentos canónicos y planes activos mezclan estado estable, transición ya terminada e historia de ejecución; algunas afirmaciones se contradicen entre Product, API e iOS. | `INCUMPLIDA` |
| `BRI-LEGAL-001` | Conservar filas estructurales tras la baja está funcionalmente decidido, pero su finalidad, base, plazo, acceso y riesgo de reidentificación continúan pendientes de aprobación Legal. | `DECISIÓN LEGAL` |
| `BRI-PLAN-001` | **Resuelto en `BRI-00C` (2026-09-19).** Existían tres planes locales activos no autorizados: `account-rule-compliance`, `crush-rule-compliance` y `crush-age-birthday` (este último con su carpeta maestra en Product vacía y una regla canónica citada inexistente). El usuario decidió: adoptar formalmente `account-rule-compliance` y `crush-rule-compliance` (añadidos a `AGENTS.md` §3 y a "Documentación permitida" de `api/AGENTS.md`); eliminar `crush-age-birthday` en ambos repos, trasladando sus hechos estables ya desplegados en producción (`age_as_of`/`zodiac_id`) a la sección "Antecedente" de `crush-rule-compliance/README.md`. | `CONTRADICCIÓN DOCUMENTAL` — resuelta; los correctivos de contenido de `account-rule-compliance`/`crush-rule-compliance` siguen bloqueados hasta el resto de `BRI-00C` |
| `BRI-ACC-001` | `CrushResource`/`PostCrushRequest`/`UpdateCrushRequest` exigen, persisten y devuelven `gender_id` de Crush en la API funcional, e iOS (`CrushEditViewModel.buildPayload`, `CrushFormModel.toAPIRequest`) lo envía activamente. Contradice `RP-CRU-001` en ambos extremos, no sólo en API. | `INCUMPLIDA` |
| `BRI-ACC-002` | `age_as_of` de Crush es aceptado como entrada de cliente en `PostCrushRequest`/`UpdateCrushRequest` (`before_or_equal:tomorrow`, admite un valor futuro) y `UpdateCrushUseCase::execute()` persiste literalmente el valor recibido del comando en vez de generarlo siempre en servidor. Contradice `RP-AGE-002`, `RP-AGE-011` y `RP-CRU-022`, que exigen que sólo la API genere `age_as_of` y rechace cualquier intento del cliente de enviarlo. | `INCUMPLIDA` |
| `BRI-ACC-003` | Un test comprometido en `api/tests/Feature/Identity/AccountMinimalDataTest.php` (`registering_persists_only_the_birth_year_in_durable_state_never_day_or_month`, `profile_view_never_exposes_full_birth_date_exact_age_name_or_phone`) exige un contrato `birth_year`-only que contradice tanto `RP-PER-002`/`RP-PER-010` (fecha completa de nacimiento en el perfil) como el contrato que el propio plan `user-information` fija como objetivo (conservar `birth_date` completo, nunca `birth_year`). `api/docs/plans/active/account-rule-compliance/README.md` ya identificó y marcó como obsoletos esos mismos tests en su fase `API-ACCOUNT-01`, pendientes de sustitución. | `CONTRADICCIÓN DOCUMENTAL` |

## 6. Referencia al catálogo único

Las reglas se leen en los documentos por dominio enlazados desde [PRODUCT.md](../../../PRODUCT.md). El inventario anterior de 128 entradas agrupadas queda sustituido por reglas identificadas individualmente, sin estados de cumplimiento mezclados con su definición.

| Alcance de comprobación futura | Documento de reglas |
|---|---|
| Identidad, fechas y alcance | [Índice y principios](../../../PRODUCT.md) |
| Cuenta, email, Apple, sesión y recuperación de acceso | [Cuenta](../../../rules/ACCOUNT.md) |
| Crush, perfil, edad, fotos y mapa | [Crush](../../../rules/CRUSH.md) |
| Experiencia inicial y posteriores | [Experiencias](../../../rules/EXPERIENCES.md) |
| Períodos, trayectos, hitos y cierres | [Relaciones](../../../rules/RELATIONSHIPS.md) |
| Timeline y Home | [Timeline](../../../rules/TIMELINE.md) |
| Guardado, conflictos y reintentos | [Sincronización](../../../rules/SYNC.md) |
| Vault y recuperación | [Vault](../../../rules/VAULT.md) |
| Estadísticas e IA | [Estadísticas](../../../rules/STATISTICS.md) |
| Tutoriales, formularios y accesibilidad | [Uso de la aplicación](../../../rules/UX.md) |
| Baja, inactividad, tareas automáticas y retención | [Ciclo de vida](../../../rules/DATA_LIFECYCLE.md) |
| Privacidad, seguridad y derechos | [Privacidad](../../../rules/PRIVACY.md) |

La matriz de Account y Crush ya está iniciada (ver más abajo); el resto de dominios sigue sin matriz. Los hallazgos previos de §5 se conservan como entradas de investigación.

## Matriz BRI-00B — Account y Crush (subalcance autorizado)

**Método y límites de esta pasada.** Evidencia obtenida mediante `codebase-memory-mcp` sobre `Users-santo-Sites-mdl-api` (HEAD `b20df4b…`, generación de índice `2026-09-17T18:12:03Z`) y `Users-santo-Sites-mdl-ios` (HEAD `93e925a…`, generación `2026-09-17T10:24:57Z`); ambos índices están por delante del último commit porque el watcher de fondo ya reindexó el WIP no confirmado del worktree (ver aviso de WIP abajo). `check_index_coverage` no reporta huecos sobre los símbolos citados. No se ejecutaron builds ni tests en esta sesión; donde la evidencia proviene de una ejecución de test registrada en otra sesión (`account-rule-compliance`, `crush-rule-compliance`, `crush-age-birthday`), se indica explícitamente como "evidencia heredada" y no se reclasifica a `CONFORME` sin contraste directo del símbolo de producción citado.

**Aviso de WIP concurrente.** Ambos worktrees tienen cambios sin commitear de `relationship-redesign` (API: `CreateCrushUseCase`, `CreatePeriodUseCase`, `UpdatePeriodUseCase`, etc.; iOS: `CrushFormModel`, `EditCrushFormModel`, `CrushEditViewModel`, pasos de formulario de Crush). El grafo indexa ese WIP porque el watcher no distingue commit de working tree. Las filas de esta matriz que tocan alta/edición de Crush pueden moverse cuando ese WIP se cierre o se revierta; se marca `(WIP)` donde aplica.

**Planes locales adoptados.** `api/docs/plans/active/account-rule-compliance/README.md` y `api/docs/plans/active/crush-rule-compliance/README.md` ya contienen una auditoría RED por regla para buena parte de Account/Crush en API, con comandos de test ejecutados y registrados; ambos quedaron adoptados formalmente en `AGENTS.md` §3 y `api/AGENTS.md` "Documentación permitida" por decisión del usuario en `BRI-00C` (ver `BRI-PLAN-001`, resuelto). El tercer plan huérfano, `crush-age-birthday`, se eliminó; su contrato `age`/`age_as_of`/`zodiac_id` ya desplegado en producción quedó documentado como antecedente dentro de `crush-rule-compliance/README.md`. Esta matriz reutiliza la evidencia de los dos planes adoptados citándola, sin duplicar su texto ni asumir que sus fases de implementación ya están aceptadas.

### Account

#### Requisitos e inventario

| ID | Aplicabilidad | Autoridad API | Preflight/observable iOS | Estado | Corrección mínima / propietario |
|---|---|---|---|---|---|
| `RP-BASE-001` | Ambas | `RegisterRequest::rules()` (`birth_date` `before_or_equal: now()->subYears(18)`), `AppleRegisterRequest::rules()` idéntico; test `AuthTest::user_can_register_on_the_exact_eighteenth_birthday` verde | `RegisterBirthdateStepView.isBirthdateValid` bloquea antes de enviar | `CONFORME` | — |
| `RP-CUE-005` | API | Inventario cerrado contrastado contra `app/Models/User.php`: `email`, `password`, `plan_id`, `country_id`, `is_tutorial_done`, `is_profile_tutorial_done`, `last_login`, `inactivity_notified_at`, `gender_id`, `birth_date`, `age` en `$fillable`. `phone` (mutator `Crypt::encryptString`) y `age` **no están en el inventario de RP-CUE-005** y siguen physically presentes/escribibles a bajo nivel aunque `UpdateUserProfileUseCase`/`UpdateUserRequest` ya los excluyan del contrato HTTP (`AccountMinimalDataTest::legacy_phone_field_is_no_longer_persisted_via_profile_update`, verde) | — | `PARCIAL` | Columnas inertes (`phone`, `age`, mutator) pendientes de retirada física; propietario `user-information` (API-UI-02/03), no BRI |
| `RP-CUE-006` | API | `UpdateUserRequest`/`UserController::update()` sólo pasan `$request->validated()`; `AccountMinimalDataTest::legacy_name_and_age_fields_are_ignored_by_profile_update_and_never_persisted` verde | — | `CONFORME` | — |
| `RP-CUE-007` | Ambas | Referencia a otros dominios (Crush/Vault/experiencias); no redefine su propio contrato aquí | — | `NO APLICABLE` (se audita en sus propios dominios) | — |

#### Registro e inicio de sesión

Autoridad común: `RegisterRequest::rules()`, `RegisterUserUseCase::execute()`, `AuthController::login()`. Evidencia fresca de esta sesión salvo donde se indica "heredada".

| ID | Aplicabilidad | Estado | Evidencia / corrección |
|---|---|---|---|
| `BR-3.01` | API | `PARCIAL` | Campos exigidos correctos en `RegisterRequest`, pero `PATCH /api/users` acepta hoy `email`/`password`/`plan_id` fuera de su flujo propio (ver `RP-PER-004`); heredado de `account-rule-compliance` (`ProfileAllowlistTest`, RED). Propietario: `API-ACCOUNT-05` |
| `BR-3.02` | API | `CONFORME` | `terms_version`/`privacy_version` validados contra `is_active` en `RegisterRequest::rules()` |
| `BR-3.03` | Ambas | `PARCIAL` | `normalizeEmail()` existe (`NormalizesTextData`) pero no se aplica de forma uniforme en recovery/cambio de email/resolución Apple (heredado, `EmailCaseInsensitivityTest` parcialmente RED). Propietario: `API-ACCOUNT-04` |
| `BR-3.04` | API | `CONFORME` | `RegisterRequest::rules()`: `min:8` + 4 regex exactos, incluida la lista `@$!%*#?&` |
| `BR-3.05` | API | `PARCIAL` | Enforcement real ya verde (`AuthTest::a_token_older_than_seven_days_is_rejected_with_401`, vía `config('sanctum.expiration')`), pero `AuthController::login()` llama `$user->createToken('auth_token')` sin `$expiresAt` explícito → `tokenExpiresAt` de la respuesta queda `null` (confirmado leyendo `login()`); heredado: actividad no probada hasta `AuthTest::authenticated_activity_does_not_extend_the_token_expiration` (RED). Propietario: `API-ACCOUNT-05` |
| `RP-CUE-001` | Ambas | `CONTRADICCIÓN DOCUMENTAL` | Ver `BRI-ACC-003`: un test comprometido pide `birth_year`-only; el contrato vigente (`RP-PER-002`, `user-information`) exige `birth_date` completo. `RegisterUserUseCase` sí persiste `birth_date` completo hoy. No corregir el código hacia `birth_year`; sustituir el test según `account-rule-compliance` |
| `RP-CUE-002` | API | `INCUMPLIDA` (heredada) | `account-rule-compliance` demuestra con test nuevo que `LoginUseCase::execute()` resetea `inactivity_notified_at` antes de `Auth::attempt()`, incluso con password incorrecta. No recontrastado línea a línea en esta sesión. Propietario: `API-ACCOUNT-05` |
| `RP-CUE-003` | Ambas | `NO DEMOSTRADA` | Middleware de sesión existe (`EnsureEmailIsVerifiedMiddlewareTest`) pero no se trazó en esta sesión el guard específico de sesión vencida/revocada fuera del caso de verificación |
| `RP-CUE-004` | iOS | `NO DEMOSTRADA` | No se trazó en esta sesión el limpiado de contexto privado en memoria al hacer logout (candidato: `AuthViewModel`/`DependencyContainer`); existe evidencia adyacente en `DependencyContainerAccountDeletionCleanupTests` para baja de cuenta, no para logout ordinario |
| `RP-CUE-008` | Ambas | `CONFORME` | `password` no reversible (`'password' => 'hashed'` en `User::casts()`); ninguna Resource devuelve `password` (`$hidden`) |

#### Verificación obligatoria por email

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `BR-3.12` | `PARCIAL` | Middleware existe y ya tiene matriz de tests (`EnsureEmailIsVerifiedMiddlewareTest`, `EnsureProvisionalAccountScopeMiddlewareTest`); heredado: el propio plan `account-rule-compliance` registra que el comportamiento correcto depende hoy de flags apagados por defecto, no de una invariante incondicional — el flag en sí es compatibilidad histórica pendiente de retirar (§7) |
| `RP-VER-001` | `CONFORME` | `EmailVerificationFlowTest::send_code_creates_an_otp_and_emails_it`; `ensureVerificationCode()` no bloquea login ante fallo de envío (`AuthController::login()`) |
| `RP-VER-002` | `NO DEMOSTRADA` | `EMAIL_VERIFICATION_OTP_EXPIRY_MINUTES` existe; cooldown progresivo e intentos/bloqueo de 15 min no se contrastaron símbolo a símbolo en esta sesión |
| `BR-3.13` | `CONFORME` | `EMAIL_VERIFICATION_PROVISIONAL_ACCOUNT_DAYS`; test `ProvisionalAccountDeadlineTest::registration_sets_a_fixed_seven_day_provisional_deadline` y `resending_the_verification_code_does_not_change_the_deadline`, ambos verdes |
| `BR-3.14` | `PARCIAL` | Mismo matiz de flags que `BR-3.12` |
| `RP-VER-003` | `PARCIAL` | Mismo matiz de flags que `BR-3.12` |
| `BR-3.15` | `CONFORME` | `ProcessProvisionalAccountExpirationsCommandTest`: baja íntegra, libera email, sin token/Vault/lead (heredado, verde) |
| `RP-VER-005` | `CONFORME` | `AppleRegisterTest::it_creates_a_verified_apple_account_and_returns_a_session` y `no_otp_is_sent_and_no_verification_row_is_created_for_a_verified_apple_signup`, verdes |

#### Acceso con Apple — alta y vinculación

Autoridad: `AppleAuthController`, `RegisterAppleUserUseCase`, `FederatedAuthContinuationService`, `AppleRegisterRequest`. iOS: `AuthViewModelAppleTests`, `AppleSignInViewModelTests`, `AppleLinkView`.

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-APL-001` | `CONFORME` | `AppleAuthController::register()`/continuación; cubierto por `AppleRegisterTest`/`AppleAuthorizeTest` |
| `RP-APL-002` | `CONFORME` | Flujo de autorización Apple obligatorio antes de continuar (`AppleAuthorizeTest`) |
| `RP-APL-003` | `NO DEMOSTRADA` | Constraint 1:1 Apple↔MDL no se contrastó en esta sesión a nivel de schema/repositorio |
| `RP-APL-004` | `CONFORME` | `AppleSignInViewModelTests` cubre fallos de autorización sin crear sesión |
| `RP-APL-005` | `CONFORME` | `AppleRegisterRequest::rules()`: `birth_date` obligatorio, mismo cálculo de 18 años que `RegisterRequest`; sin `password` |
| `RP-APL-006` | `NO DEMOSTRADA` | No se trazó en esta sesión la selección personal/intermediario de Apple símbolo a símbolo |
| `RP-APL-008` | `NO DEMOSTRADA` | No trazado en esta sesión |
| `RP-APL-009` | `NO DEMOSTRADA` | No trazado en esta sesión (ausencia de captura de `fullName` de Apple) |
| `RP-APL-010` | `NO DEMOSTRADA` | Ventana de 10 minutos de `continuation_token`; `AppleRegisterRequest` valida `size:43` pero no se contrastó el TTL en esta sesión |
| `RP-APL-011` | `NO DEMOSTRADA` | Atomicidad del alta compuesta Apple no recontrastada en esta sesión |
| `RP-APL-012` | `NO DEMOSTRADA` | No trazado |
| `RP-APL-014` | `NO DEMOSTRADA` | Aviso previo de uso del email de Apple; UI no trazada en esta sesión |
| `RP-APL-024` | `NO DEMOSTRADA` | No trazado |
| `RP-APL-017` | `CONFORME` | `UserTest::a_password_account_reports_password_as_its_only_auth_method` y simétrico Apple; ambos métodos conviven en `User` |
| `RP-APL-018` | `INCUMPLIDA` (heredada) | `account-rule-compliance`: no existe ruta autenticada dedicada para vincular Apple desde sesión abierta (`RP-APL-030`); el único camino hoy pasa por el flujo no autenticado. Propietario: `API-ACCOUNT-07` |
| `RP-APL-015` | `INCUMPLIDA` (heredada) | No existe flujo autenticado para que una cuenta Apple añada su primera contraseña; `account-rule-compliance` lo deja para `API-ACCOUNT-07` |
| `RP-APL-016` | `INCUMPLIDA` (heredada) | Depende del mismo flujo ausente que `RP-APL-015` |
| `RP-APL-026` | `NO DEMOSTRADA` | Depende de que exista el flujo de `RP-APL-015`; sin ese flujo, la regla no es contrastable todavía |
| `RP-APL-032` | `INCUMPLIDA` (heredada) | Mismo flujo ausente; `RP-OTP-007` (cierre de sesiones) tampoco está atómico hoy (ver abajo) |
| `RP-APL-027` | `NO DEMOSTRADA` | No trazado en esta sesión |
| `RP-APL-028` | `NO DEMOSTRADA` | No trazado |
| `RP-APL-029` | `NO DEMOSTRADA` | No trazado |
| `RP-APL-030` | `INCUMPLIDA` (heredada) | `account-rule-compliance`: "No existe vínculo Apple desde una sesión autenticada." Propietario: `API-ACCOUNT-07` (`sign-in-with-apple` es la iniciativa de producto antecedente; el correctivo ejecutable vive en `account-rule-compliance`/BRI, no debe duplicarse en `sign-in-with-apple`) |
| `RP-APL-031` | `NO DEMOSTRADA` | No trazado |
| `RP-APL-033` | `NO DEMOSTRADA` | `account-rule-compliance` la deja pendiente de evidencia conjunta en `API-ACCOUNT-03` |
| `RP-APL-020` | `NO DEMOSTRADA` | Revocación de autorización Apple y cierre de sesiones asociadas no trazado en esta sesión |
| `RP-APL-023` | `CONFORME` (iOS) | `AppleAuthorizationErrorMapper.map` + manejo en `AuthViewModelAppleTests`/`AppleSignInViewModelTests` para fallos de disponibilidad |

#### Perfil de la cuenta de usuario

Autoridad: `UpdateUserRequest`, `UserController::update()`, `UserResource::toArray()`.

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-PER-001` | `PARCIAL` | Omitir conserva (heredado, sin test dedicado a "borrado explícito" porque ningún campo actual lo permite aún); `ProfileAllowlistTest` cubre el resto |
| `RP-PER-002` | `CONTRADICCIÓN DOCUMENTAL` | Ver `BRI-ACC-003`. `UserResource::toArray()` hoy expone `birth_date` completo (no `birthYear`), lo que de hecho coincide con la regla vigente; el riesgo está en el test comprometido, no en la Resource actual |
| `RP-PER-003` | `NO DEMOSTRADA` | Preferencia de idioma local; no trazada en esta sesión |
| `RP-PER-004` | `INCUMPLIDA` (heredada) | `ProfileAllowlistTest`: `PATCH /api/users` acepta hoy `email`/`password`/`plan_id` con valor válido y muta. Propietario: `API-ACCOUNT-05` |
| `RP-PER-007` | `CONFORME` | `UserResource::toArray()` devuelve `gender_id`/`country_id`; `AccountMinimalDataTest::profile_view_never_exposes_full_birth_date_exact_age_name_or_phone` confirma que siguen presentes como catálogo |
| `RP-PER-008` | `PARCIAL` | Nombre/edad ya excluidos del contrato HTTP (verde); `phone` ya no se persiste vía perfil (verde) pero la columna/mutator siguen vivos a nivel de modelo — ver `RP-CUE-005`. Propietario `user-information` para el retiro físico |
| `RP-PER-009` | `PARCIAL` | Mismo hallazgo que `RP-PER-004`: la allowlist HTTP no es todavía estricta contra email/password/plan |
| `RP-PER-010` | `CONFORME` | `UserResource::toArray()` expone `birth_date`, `gender_id`, `country_id`; no expone edad exacta calculada |

#### Protección del email, cambio de email y contraseña

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-VLT-005` | `INCUMPLIDA` | Confirmado leyendo `app/Models/User.php`: `email` no tiene cast de cifrado ni mutator; `search_code` no encuentra `email_lookup` en `app/`. Coincide con `ARCHITECTURE.md` §10.1 ("cifrado de campo y búsqueda pendientes en `user-information`"). Propietario: `user-information` (API-UI-00/01), no BRI |
| `RP-CUE-009` | `INCUMPLIDA` | Mismo hallazgo: sin cifrado de campo, ninguna de las copias enumeradas está protegida hoy. Propietario: `user-information` |
| `RP-MAIL-001` | `NO DEMOSTRADA` | No trazado en esta sesión |
| `RP-MAIL-002` | `NO DEMOSTRADA` | No trazado |
| `RP-MAIL-003` | `NO DEMOSTRADA` | No trazado |
| `RP-MAIL-004` | `NO DEMOSTRADA` | No trazado |
| `RP-MAIL-005` | `NO DEMOSTRADA` (heredada) | `account-rule-compliance` la deja pendiente de evidencia conjunta con `RP-APL-033` en `API-ACCOUNT-03` |
| `RP-PWD-001` | `INCUMPLIDA` (heredada) | No existe flujo autenticado dedicado de cambio de contraseña con contraseña actual obligatoria; único camino hoy es recovery sin sesión. Propietario: `API-ACCOUNT-06` |
| `RP-PWD-002` | `NO DEMOSTRADA` | Depende del flujo ausente de `RP-PWD-001` |
| `RP-PWD-003` | `INCUMPLIDA` (heredada) | Depende de `RP-PWD-001` y de `RP-OTP-007` (revocación atómica), ambos pendientes |
| `RP-PWD-004` | `NO APLICABLE hoy / PARCIAL` | Coherente con `RP-APL-015`: al no existir el flujo de alta de contraseña para cuentas Apple, tampoco hay riesgo de que este flujo se use indebidamente, pero la exclusión positiva no está probada |
| `BR-3.06` | `NO DEMOSTRADA` | No trazado en esta sesión (anti-enumeración de recovery) |
| `RP-OTP-001` | `NO DEMOSTRADA` | Consumo único bajo concurrencia no trazado en esta sesión |
| `BR-3.07` | `NO DEMOSTRADA` | No trazado |
| `BR-3.08` | `NO DEMOSTRADA` | No trazado |
| `BR-3.16` | `NO DEMOSTRADA` | No trazado |
| `RP-OTP-005` | `NO DEMOSTRADA` | No trazado |
| `RP-OTP-007` | `INCUMPLIDA` (heredada) | `account-rule-compliance`: "Cambiar/restablecer password no revoca todos los tokens" — bloquea también `RP-PWD-003`/`RP-APL-032`. Propietario: `API-ACCOUNT-06` |

#### Bloqueo local (iOS)

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-LOCK-001` | `CONFORME` | `AppLockAttemptCoordinatorTests`, `AppLockOverlaySuppressionStateTests` cubren bloqueo al volver de background, incluida superposición sobre otra pantalla |
| `RP-LOCK-002` | `CONFORME` | Bloqueo local no toca sesión/Vault: `VaultSessionIntegrationTests.testBiometricLoginActivatesVaultBeforeUnlocking` separa ambos conceptos |

### Crush

#### Inventario y alta

Autoridad API: `PostCrushRequest::rules()`, `UpdateCrushRequest::rules()`, `CrushResource::toArray()`, `CreateCrushUseCase`/`UpdateCrushUseCase` (WIP de `relationship-redesign` en curso sobre estos mismos archivos). iOS: `CrushFormModel`, `EditCrushFormModel`, `CrushEditViewModel.buildPayload`.

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-CRU-013` | `PARCIAL` | El inventario general coincide, salvo la excepción confirmada de `gender_id` (ver `RP-CRU-001`) |
| `RP-CRU-014` | `CONFORME` | `height_id`/`eye_color_id`/`hair_color_id`/`body_type_id`/`skin_color_id` viajan como enteros de catálogo en `UpdateCrushRequest`/`CrushResource`, nunca texto libre |
| `RP-CRU-015` | `PARCIAL` | Mismo matiz que `RP-CRU-013` por `gender_id` |
| `BR-4.01` | `PARCIAL` | Campos exigidos presentes en `PostCrushRequest::rules()` (`ducky_id`, `age`, `country_id`, `met_at`, `lookup_token`), pero `age_as_of` es `sometimes` en vez de generado server-side (ver `BRI-ACC-002`) y `gender_id` es `required` cuando debería vivir sólo en Vault (ver `RP-CRU-001`) |
| `RP-CRU-004` | `NO DEMOSTRADA` | Atomicidad del alta compuesta (Crush + período inicial + experiencia inicial) no recontrastada en esta sesión; además coincide con el WIP no commiteado de `relationship-redesign` sobre `CreateCrushUseCase`/`CreatePeriodUseCase` |
| `RP-CRU-005` | `INCUMPLIDA` (API) / `CONFORME` (iOS) | `PostCrushRequest`/`UpdateCrushRequest`: `ducky_id` sólo `integer`, sin `Rule::in([1,2,3,4])` — confirmado leyendo ambos Requests. iOS: `enum CrushDucky: Int, CaseIterable { casualDuck=1...uglyDuck }` restringe correctamente la UI. Propietario API: `API-CRU-01` (`crush-rule-compliance`) |

#### Perfil — contenido exclusivo del Vault

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-CRU-001` | `INCUMPLIDA` | Confirmado en ambos extremos: `PostCrushRequest::rules()` → `'gender_id' => ['required','integer', GenderExists]`; `UpdateCrushRequest::rules()` → `'gender_id' => 'sometimes\|integer'`; `CrushResource::toArray()` → `'gender_id' => $this->gender_id`. iOS `CrushEditViewModel.buildPayload()` línea `parameters["gender_id"] = formData.gender.rawValue` lo envía activamente. Ver `BRI-ACC-001`. Propietario: `API-CRU-01` + fase iOS equivalente (`IOS-BRI-01` o fase dedicada, a definir en `BRI-00C`) |
| `RP-CRU-012` | `PARCIAL` | Profesión/WhatsApp/redes no se trazaron símbolo a símbolo en la API en esta sesión (evidencia heredada de `crush-rule-compliance`: "acepta, persiste y devuelve detalles privados en claro") |
| `RP-CRU-023` | `NO DEMOSTRADA` | Límites de longitud (191/255/20) no recontrastados en esta sesión |

#### Rasgos físicos estructurados

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-CRU-016` | `CONFORME` | Cinco campos opcionales independientes, enteros de catálogo, confirmados en `UpdateCrushRequest::rules()` |
| `RP-CRU-017` | `NO DEMOSTRADA` | Independencia mutua entre rasgos y el resto del perfil no trazada explícitamente en esta sesión |
| `RP-CRU-018` | `NO DEMOSTRADA` | Semántica de `null` explícito para "quitar" un rasgo no trazada en esta sesión (distinto del triestado de `zodiac_id`, que sí está confirmado, ver `RP-AGE` abajo) |

#### Lectura, búsqueda y edición del perfil

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-CRU-019` | `CONFORME` (iOS) | `VaultUIManagerTests`, `AppLock*Tests` cubren bloqueo del Vault al iniciar/volver de background sobre listas/mapa/Timeline/fichas |
| `RP-CRU-020` | `NO DEMOSTRADA` | Composición ficha API+Vault no recontrastada en esta sesión |
| `RP-CRU-021` | `NO DEMOSTRADA` | Búsqueda privada local (sin envío del término a la API) no trazada en esta sesión |
| `RP-CRU-022` | `INCUMPLIDA` | Ver `RP-CRU-001` (gender_id editable) y `BRI-ACC-002` (age_as_of aceptado del cliente). El triestado de `zodiac_id` sí está confirmado: `UpdateCrushCommand::hasZodiacId` + `CrushEntity::updateZodiac(?int)`, evidencia de despliegue documentada como antecedente en `api/docs/plans/active/crush-rule-compliance/README.md` (ex `API-AGE-C1`, heredada, sin resultado de suite registrado todavía) |

#### Fecha de encuentro

Autoridad: `UpdateCrushUseCase::handleMetAtUpdate()`/`validateChronologicalIntegrity()`, regla `CrushMetAtBeforeEntities`.

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-CRU-007` | `PARCIAL` | `PostCrushRequest`: `met_at` con `before_or_equal:today` (sin fecha futura, correcto). El límite frente al primer acontecimiento independiente es el hallazgo de `RP-CRU-008` |
| `RP-CRU-008` | `INCUMPLIDA` | Confirmado leyendo `validateChronologicalIntegrity()`: rechaza sólo si `experience->date() < newMetAt` (estrictamente anterior), por lo que un acontecimiento independiente con la **misma fecha** que el nuevo `met_at` no se rechaza — el contrato exige estrictamente anterior al primero de esos acontecimientos, no igual. Propietario: `API-CRU-02` (`crush-rule-compliance`) |
| `RP-CRU-009` | `INCUMPLIDA` | Confirmado leyendo `handleMetAtUpdate()`: `crush->save()`, `bundledPeriod->save()` y `bundledExperience->save()` son tres escrituras separadas sin `DB::transaction`/lock; un fallo intermedio deja el bundle desincronizado. Propietario: `API-CRU-02` |

#### Valoraciones

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `BR-4.02` | `INCUMPLIDA` | Confirmado leyendo `UpdateCrushRequest::rules()`: `rate_beauty`/`rate_emotional_connection`/`rate_personality`/`rate_sex_appeal` son `numeric\|between:1,10` (admite decimales y hasta 10, no enteros 1–5) y `rate_global` es `sometimes\|nullable\|numeric\|between:0,10` (el cliente puede enviarlo, cuando el contrato exige que sólo lo calcule el servidor). `UpdateCrushUseCase::handleRateUpdate()` sí recalcula la media cuando cambia un rate individual, pero no evita que `rate_global` llegue como entrada directa. Propietario: `API-CRU-01` |

#### Edad, cumpleaños y zodíaco

Autoridad: `UpdateCrushCommand`/`CrushEntity` (triestado); antecedente de despliegue documentado en `api/docs/plans/active/crush-rule-compliance/README.md` (ex `API-AGE-01/02/C1`, ya desplegadas en producción).

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-AGE-001` | `CONFORME` | `age` validado `between:18,80` en alta y edición |
| `RP-AGE-002` | `INCUMPLIDA` | Ver `BRI-ACC-002`: el cliente puede enviar `age_as_of` y la API lo persiste tal cual en vez de generarlo siempre server-side; además `before_or_equal:tomorrow` admite un valor futuro |
| `RP-AGE-003` | `NO APLICABLE a esta matriz API/iOS estructural` | Vive en Vault (payload `birthdate`); ver `RP-CRU-012` |
| `RP-AGE-004` | `NO DEMOSTRADA` | Derivación de `zodiac_id` desde día/mes no trazada en esta sesión (vive en iOS, contenido Vault) |
| `RP-AGE-005` | `NO DEMOSTRADA` | No trazado en esta sesión |
| `RP-AGE-006`–`RP-AGE-009` | `NO DEMOSTRADA` | Cálculo de edad mostrada (intervalos, 29 feb) no trazado en esta sesión; el antecedente en `crush-rule-compliance/README.md` no cubre esta parte, es responsabilidad de estadísticas/presentación |
| `RP-AGE-010` | `NO DEMOSTRADA` | No trazado |
| `RP-AGE-011` | `INCUMPLIDA` | Mismo hallazgo que `RP-AGE-002`: la API no rechaza `age_as_of` de cliente |
| `RP-AGE-012` | `NO DEMOSTRADA` | No trazado (vive mayormente en Vault/iOS) |
| `RP-AGE-013` | `NO DEMOSTRADA` | No trazado |
| `RP-AGE-014` | `NO DEMOSTRADA` | No trazado |
| `RP-AGE-015` | `PARCIAL` | Coherente con el contrato de pareja documentado como antecedente en `crush-rule-compliance` (`PATCH sin edad conserva pareja`, `PATCH con pareja actualiza ambos`), pero no hay todavía una ejecución de suite fresca registrada (ex `API-AGE-C1`: "no conserva aún un resultado literal de test run") |

#### Fotos del Crush (iOS)

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-FOT-001` | `CONFORME` | `LocalMediaManager`, sin subida a Vault/servidor para fotos de Crush en la beta |
| `RP-FOT-002` | `CONFORME` | `EXIFStripper.stripEXIF()`/`verifyNoEXIF()` con tests dedicados |
| `RP-FOT-003` | `CONFORME` | `LocalMediaManagerTests::testSavedPhotoFileIsExcludedFromBackup`, `testPerCrushDirectoryIsExcludedFromBackup`, `testRootLocalMediaDirectoryIsExcludedFromBackup` |
| `RP-FOT-004` | `CONFORME` | `LocalMediaManagerTests::testDeletingASinglePhotoStillRemovesOnlyThatFile` |
| `RP-FOT-005` | `CONFORME` | `LocalMediaManagerTests::testDeleteAllMediaStillRemovesTheWholeCrushDirectory`; baja de cuenta cubierta por `DependencyContainerAccountDeletionCleanupTests::testDirect2xxWipesSQLiteKeychainLocalMediaAndOnboardingState` |
| `RP-FOT-006` | `NO DEMOSTRADA` | No trazado en esta sesión |
| `RP-FOT-007` | `PARCIAL` | `PhotoPickerCrush` fija `PHPickerConfiguration.selectionLimit = 10` (límite de selección por lote, no el tope de 5 por Crush); el tope de 5 y la liberación de hueco al borrar no se localizaron en esta sesión en el punto que agrega selección + fotos ya guardadas — falta trazar `CrushViewModel.uploadMedia`/`viewModel.uploadMedia` |
| `RP-FOT-008` | `PARCIAL` | `ImageConversionHelper.handleLargeImage()` **comprime/recomprime** imágenes grandes en vez de rechazarlas por encima de 10 MB; el contrato pide rechazo explícito sin alterar las fotos ya guardadas. Falta confirmar si existe un rechazo duro tras la compresión agresiva o si siempre se acepta un resultado comprimido |

#### Mapa de Crushes por país (iOS)

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-MAP-001`–`RP-MAP-005` | `NO DEMOSTRADA` | `MapViewModel.loadMapItems`, `MapCountryDetailsModel` existen y tienen tests de datos (`MapData.mockMapData`), pero esta sesión no trazó símbolo a símbolo exclusión de archivados/eliminados, ventana única de info ni conservación de filtros al volver de la lista |

#### Archivar y restaurar

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-ARC-001` | `NO DEMOSTRADA` | No trazado en esta sesión |
| `RP-ARC-002` | `NO DEMOSTRADA` | No trazado |
| `RP-ARC-003` | `NO DEMOSTRADA` | No trazado |

#### Eliminar un Crush

Autoridad: `VaultCrushDeletionCoordinator` (iOS), coordinación con baja funcional de API (no trazada específicamente para Crush individual en esta sesión, a diferencia de la baja de cuenta cubierta en `BRI-DEL-*`).

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-DEL-001` | `NO DEMOSTRADA` | No se trazó en esta sesión el endpoint de borrado funcional de Crush individual ni su alcance sobre experiencias/períodos/hitos/Vault/fotos en un solo contraste |
| `RP-DEL-002` | `NO DEMOSTRADA` | Distinción frente a la conservación estructural de `DATA_LIFECYCLE.md` no trazada en esta sesión para Crush individual |
| `RP-DEL-003` | `PARCIAL` (iOS) | `VaultCrushDeletionCoordinatorTests` existe con fixtures de medios locales; la persistencia durable de la intención de borrado antes de llamar al servidor no se confirmó símbolo a símbolo en esta sesión |
| `RP-DEL-004` | `PARCIAL` (iOS) | `LocalMediaManagerTests::testDeleteAllMediaOnAnAlreadyAbsentCrushIsIdempotent` da evidencia de idempotencia en el lado local; la semántica exacta de "no encontrado sólo confirma tras un intento registrado" no se trazó en la API en esta sesión |
| `RP-DEL-005` | `NO DEMOSTRADA` | Orden borrado-funcional-primero-luego-local no trazado símbolo a símbolo en esta sesión para Crush individual (sí está demostrado para baja de cuenta completa en los hallazgos `BRI-DEL-*` de §5) |

**Resumen de estados — Account (76 IDs, 77 filas de matriz).** `CONFORME`: 19 · `PARCIAL`: 10 · `INCUMPLIDA`: 12 · `NO DEMOSTRADA`: 32 · `CONTRADICCIÓN DOCUMENTAL`: 2 · `NO APLICABLE`/mixto: 2. El recuento es por fila, no por ID individual (alguna fila anota un matiz "PARCIAL" adicional dentro de un estado principal); no busca cuadrar aritméticamente contra 76 sin revisar cada fila una a una en `BRI-00C`.

**Resumen de estados — Crush (55 IDs, filas de matriz agrupan `RP-AGE-006`–`RP-AGE-009` y `RP-MAP-001`–`RP-MAP-005` en una sola fila cada una).** `CONFORME`: 12 · `PARCIAL`: 12 · `INCUMPLIDA`: 10 · `NO DEMOSTRADA`: 23 (cubriendo 8 IDs adicionales por las dos filas agrupadas) · `CONTRADICCIÓN DOCUMENTAL`: 2 (hallazgos transversales `BRI-ACC-001`/`BRI-ACC-003`, que también son Crush) · `NO APLICABLE`: 3.

Ambos recuentos son un apoyo de lectura, no un criterio de cierre: la fuente de verdad es la fila individual de cada ID en las tablas de arriba.

## 7. Compatibilidad histórica que debe auditarse y retirarse

La búsqueda debe cubrir como mínimo:

- `legacy`, `old`, `previous`, `migration`, `migrate`, `fallback`, `compat`, `v1`, `state_id`, `RelationshipState`, `match_status_id`, `secure_description` y campos privados retirados;
- gates por fecha de alta, versión de cuenta, existencia/ausencia de Vault o forma antigua de snapshot;
- rutas de migración Vault, planners/comandos de migración relacional y superficies Debug asociadas;
- upgraders/decoders y defaults que acepten claves antiguas;
- operaciones durables “legacy” y recuperación de scope a partir de snapshots ambiguos;
- fixtures, mocks y tests que exijan ambos contratos;
- jobs/placeholders de cleanup y columnas inertes que todavía influyan en runtime;
- documentos que describan el usuario histórico como dependencia futura.

Cada coincidencia se clasifica como:

1. retirar de inmediato;
2. mecanismo actual no legacy mal nombrado;
3. evidencia histórica que debe desaparecer de docs vigentes;
4. compatibilidad real necesaria sólo durante un rollout posterior a 1.0;
5. dato/schema inerte cuya eliminación física requiere una fase destructiva separada.

## 8. Fases

### `BRI-00A` — catálogo funcional

**Salida:** reglas por dominio en `docs/rules/`, con índice en `PRODUCT.md`, identificadores conservados y decisiones pendientes explícitas. Primero el usuario revisa y finaliza cada archivo por separado. No se evalúa cumplimiento de código ni se modifica Legal.

### `BRI-00B` — contraste de implementación

**Estado:** iniciada, limitada al subalcance autorizado Account + Crush (ambos `FINALIZADO`). Ver «Matriz BRI-00B — Account y Crush» arriba para las 131 filas (76 Account + 55 Crush) con estado, evidencia y propietario. El resto de dominios (`EXPERIENCES`, `RELATIONSHIPS`, `TIMELINE`, `SYNC`, `VAULT`, `STATISTICS`, `UX`, `DATA_LIFECYCLE`, `PRIVACY`) sigue `EN REVISIÓN` y sin contraste; `BRI-00B` no se declara completa hasta cubrirlos.

Para cada regla referenciada desde §6:

- localizar autoridad API, preflight iOS, consumidores y tests mediante índices frescos;
- registrar `CONFORME/PARCIAL/INCUMPLIDA/NO DEMOSTRADA` con símbolos y evidencia;
- recorrer todas las rutas alternativas que puedan eludir la regla;
- completar el inventario de compatibilidad del §7;
- no corregir nada todavía.

### `BRI-00C` — revisión funcional

Revisar con el usuario, una por una, las discrepancias de implementación demostradas. Las definiciones funcionales ya se revisan antes de `BRI-00B`; cualquier decisión nueva se actualiza primero en el documento de reglas del dominio, enlazado desde `PRODUCT.md`. Congelar los resultados antes de abrir fixes.

### Fases correctivas

Se crean sólo desde la matriz congelada, y sólo después de `BRI-00C`. Todas siguen **bloqueadas**; ninguna se autoriza ni empieza desde esta sesión. Candidatos ya demostrados para el resto del catálogo:

1. `API-BRI-01`: separar anonimización estructural de destrucción Vault y corregir baja de cuenta, purga, actividad y estadísticas post-baja.
2. `API-BRI-02`: retirar compatibilidad/runtime legacy de API y su cobertura.
3. `IOS-BRI-01`: retirar compatibilidad/runtime legacy de iOS y su cobertura.
4. lotes posteriores por dominio, con una regla o grupo inseparable por fase.

Candidatos específicos del subalcance Account/Crush, detallados en los planes locales de API e iOS:

5. `API-CRU-01`/`API-CRU-02` (ya definidas en `api/docs/plans/active/crush-rule-compliance/README.md`): cierre del contrato funcional del perfil de Crush (`gender_id`, `ducky_id`, ratings, `age_as_of` de cliente) y mutación atómica de `met_at`.
6. `API-ACCOUNT-04..08` (ya definidas en `api/docs/plans/active/account-rule-compliance/README.md`): identidad protegida/perfil mínimo, fronteras de perfil/login/sesión, verificación/recovery/password obligatorios, métodos Apple autenticados desde sesión abierta, integración y cierre.
7. Fase iOS dedicada (a numerar en `BRI-00C`) para dejar de enviar `gender_id`/`age_as_of` de Crush desde `CrushEditViewModel.buildPayload`/`CrushFormModel.toAPIRequest` una vez que la API deje de aceptarlos — depende de 5.

`BRI-PLAN-001` (§5) está resuelto: `account-rule-compliance`/`crush-rule-compliance` quedaron adoptados en `AGENTS.md` §3 y `api/AGENTS.md`, y `crush-age-birthday` se eliminó (su antecedente vive ahora en `crush-rule-compliance/README.md`). Las fases 5–6 ya tienen plan autorizado que las cubra; siguen bloqueadas únicamente por el resto de `BRI-00C`, igual que el resto de fases correctivas de este documento.

Los fixes relacionales todavía abiertos permanecen en `relationship-redesign` hasta su aceptación. No se duplican aquí. Los correctivos de cifrado/minimización del email y del nacimiento de cuenta (`RP-VLT-005`, `RP-CUE-009`, `RP-PER-008`, y el propio contrato `birth_date`/`birth_year` de `BRI-ACC-003`) permanecen en `user-information`, no en BRI.

### `BRI-CLOSE`

Sólo cuando todas las reglas de beta estén `CONFORME` o explícitamente `FUERA DE BETA`, las decisiones Legal bloqueantes estén resueltas y no quede compatibilidad histórica activa:

- integrar reglas estables en documentos canónicos;
- eliminar contradicciones e historia de ejecución;
- retirar este plan y los locales;
- verificar enlaces y `git diff --check` en cada repo.

## 9. Gates de `BRI-00B`

La auditoría de código no se declara completa hasta que:

- `codebase-memory-mcp` indexe de forma fresca los worktrees de API e iOS;
- se compruebe cobertura de cada ruta/símbolo citado;
- se inspeccionen consumidores, mocks, fixtures y tests de contratos compartidos;
- se distingan cambios ya aceptados de WIP simultáneo;
- no se use búsqueda textual como sustituto del grafo para relaciones estructurales.

Al iniciar el subalcance Account/Crush de `BRI-00B`, el índice API seguía en `b20df4b…` y el de iOS en `93e925a…` (mismos commits que al abrir este plan). Claude Code no reindexa por protocolo; `check_index_coverage` sobre los símbolos citados en la matriz de Account/Crush no reportó huecos, y la generación del índice (`2026-09-17T18:12:03Z` API, `2026-09-17T10:24:57Z` iOS) es posterior al commit, es decir, ya incluye el WIP sin commitear de `relationship-redesign` sobre archivos de Crush (ver aviso de WIP en la matriz). Eso permite tratar la matriz de Account/Crush como evidencia fresca del código actual, con la salvedad expresa de que las filas que tocan alta/edición de Crush pueden moverse cuando ese WIP se cierre o revierta. El resto del catálogo (fuera de Account/Crush) sigue sin esta comprobación.

## 10. Validación y disciplina de tests

- Tests dirigidos uno a uno a la regla; no suites completas por defecto.
- Mutaciones: autorización cruzada, idempotencia, rechazo sin parcialidad y rollback.
- Invariantes concurrentes: motor real que represente locks.
- Durable writes: misma identidad/snapshot tras retry y relanzamiento.
- UI: acción real, no mera presencia de alerta; fixtures equivalentes a producción.
- Gates físicos sólo cuando dependan de dispositivo, Keychain, offline/online, S3, scheduler o infraestructura desplegada.
- Ningún test se debilita para preservar compatibilidad que este plan retira.

## 11. Decisiones abiertas

1. Legal debe aprobar finalidad, base, plazo, acceso y salvaguardas de la estructura seudonimizada conservada tras la baja.
2. Reconciliar la ejecución relacional con las reglas actuales de [Relaciones](../../../rules/RELATIONSHIPS.md), incluidas las transiciones lógicas. Legal conserva revisión independiente y no se modifica desde BRI sin alcance explícito.
3. Definir si las columnas físicamente inertes se eliminan antes de 1.0 o en una operación destructiva separada; en ambos casos no pueden influir en runtime.
4. Confirmar, después del inventario, si Google auth/IAP/personality/notification placeholders tienen consumidor de beta; si no, quedan inaccesibles o se retiran según su riesgo, sin mantenerlos por “compatibilidad”.
5. ~~`BRI-PLAN-001`~~ — **Resuelto (2026-09-19).** `account-rule-compliance` y `crush-rule-compliance` adoptados en `AGENTS.md` §3 y `api/AGENTS.md`; `crush-age-birthday` eliminado, con su contrato ya desplegado documentado como antecedente en `crush-rule-compliance/README.md`. Sus fases de implementación (`API-ACCOUNT-03..08`, `API-CRU-RED-01/02`, `API-CRU-01/02`) siguen bloqueadas por el resto de `BRI-00C`, no por falta de plan autorizado.
6. `BRI-ACC-003`: resolver si el perfil de cuenta expone `birth_date` completo (texto vigente de `RP-PER-002`/`RP-PER-010` y contrato de `user-information`) o migra a `birth_year`-only (lo que ya asume un test comprometido en `api/tests/Feature/Identity/AccountMinimalDataTest.php`). Mientras no se decida, ese test no puede usarse como criterio de implementación y `user-information` sigue construyendo sobre `birth_date` completo.
