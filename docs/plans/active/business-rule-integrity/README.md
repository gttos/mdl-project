# Integridad de reglas de negocio

> **Estado:** todo el catálogo de `docs/rules/` está `FINALIZADO` por indicación expresa de Producto. `BRI-00B`/`BRI-00C` se ejecutaron únicamente para el subalcance Account y Crush: el usuario revisó esas discrepancias una por una (2026-09-19) y confirmó que se corrigen conforme a sus reglas, sin reabrirlas. Los demás dominios están finalizados funcionalmente pero todavía no han sido contrastados contra código, tests y documentación local.
>
> **Recepción coordinadora del trabajo no commiteado atribuido a Claude (2026-09-19): implementación aceptada con validaciones pendientes.** Se reindexaron `product/`, `api/`, `ios/` y `android/` sobre los worktrees actuales y se releyeron las rutas de producción principales de Crush. Las ejecuciones de tests descritas en este plan son evidencia reportada y no se repitieron en esta recepción. Sin commits que aíslen los lotes no puede demostrarse la autoría archivo por archivo. El contraste vigente identifica tres grupos todavía incumplidos: consumidores funcionales de `gender_id` en API (`RP-CRU-001`, `RP-CRU-013`, `RP-CRU-015`); corrección cronológica/atómica de `met_at` (`RP-CRU-008`, `RP-CRU-009`); y el contrato añadido en el worktree de experiencia inicial obligatoria (`BR-4.01`, `RP-CRU-004`), que API, iOS y Android todavía tratan como opcional. `RP-DEL-006`, también añadido en el worktree, es conforme en iOS y está incumplido en Android porque la acción continúa disponible dentro de la ficha.
>
> **Autorización de fixes (2026-09-19).** El usuario autorizó expresamente iniciar fases correctivas de código para todo lo que esté en contra de `ACCOUNT.md`/`CRUSH.md`: "el alcance es todo lo que esté en contra de las reglas que hemos diseñado en ACCOUNT.md y CRUSH.md, cualquier otra contradicción de texto o código que encuentres, prevalece lo que hemos puesto en estos dos MD". Estos dos documentos son la autoridad que prevalece sobre cualquier código, test o documento técnico que los contradiga en su alcance. Bajo ese mandato, el coordinador puede: iniciar fases GREEN de `account-rule-compliance`/`crush-rule-compliance` ya diseñadas y con su RED correspondiente aceptada; investigar filas `NO DEMOSTRADA` y, si confirma una violación real, corregirla; y encadenar fases dependientes (p. ej. `IOS-CRU-01` después de que `API-CRU-01` esté verde) sin volver a pedir autorización puntual por cada una.
>
> **Excluido de esta autorización, aunque esté dentro de Account/Crush** (no se ejecuta esta noche; requiere una sesión y decisión separadas):
> - `RP-VLT-005`/`RP-CUE-009` (cifrado de campo del email de cuenta): pertenece a `user-information`, que exige primero una fase `UI-00` de diseño de custodia/rotación de claves y orden de corte — no es un fix de código aislado, es una decisión de arquitectura de seguridad con migración de datos reales eventual.
> - Cualquier migración/backfill/corte aplicado sobre datos reales (dev o producción) o cualquier eliminación física de columna (`phone`, `age` inertes en `users`). Preparar y probar una migración contra la base de test (sqlite efímera) sí está permitido; aplicarla fuera de test no.
> - Cualquier decisión que reabra el texto ya `FINALIZADO` de `ACCOUNT.md`/`CRUSH.md` en vez de corregir el código hacia ese texto.
> - El gate Legal ya conocido sobre la estructura seudonimizada tras la baja (no es de Account/Crush individual, es de `DATA_LIFECYCLE.md`).
>
> El estado vigente de cada fase (no un diario cronológico) se mantiene en la tabla "Orden y dependencias"/"Orden de ejecución" de `account-rule-compliance/README.md` y `crush-rule-compliance/README.md`, sustituyendo su estado anterior a medida que cada fase se acepta — igual que ya venían haciendo esos planes.

## 1. Objetivo

El catálogo de reglas tiene su índice en [PRODUCT.md](../../../PRODUCT.md) y sus definiciones por dominio en `docs/rules/`. Producto cerró expresamente su revisión. Esta iniciativa debe comprobar ahora cada regla contra la implementación y proponer correcciones sin reinterpretar ni debilitar el catálogo. Este plan no es la fuente funcional ni mantiene reglas alternativas.

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
| `BRI-REL-001` | El documento `rules/RELATIONSHIPS.md` reúne situaciones, períodos, hitos `latest-only` y trayectos válidos; Timeline y el expediente Legal ya describen ese contrato. Queda por reconciliar la ejecución técnica y por obtener validación jurídica antes del lanzamiento; no se ha comprobado cumplimiento del código. | `CONTRADICCIÓN DE IMPLEMENTACIÓN` pendiente / contrato documental único definido |
| `BRI-REL-002` | La validación interactiva demostró que «Editar relación» podía agregar historia. `API-RR-04C` fue reportada, pero su aceptación coordinada y el correctivo iOS siguen fuera de esta auditoría hasta contar con grafo fresco y evidencia completa. | `PARCIAL` |
| `BRI-LEG-001` | Vault, durable write, relación, email y arquitectura todavía contienen rutas, modelos, fallbacks, fixtures o requisitos del “único usuario legacy”. La decisión greenfield los vuelve deuda a retirar, no compatibilidad a preservar. | `INCUMPLIDA` |
| `BRI-DOC-001` | Documentos canónicos y planes activos mezclan estado estable, transición ya terminada e historia de ejecución; algunas afirmaciones se contradicen entre Product, API e iOS. | `INCUMPLIDA` |
| `BRI-LEGAL-001` | Conservar filas estructurales tras la baja está funcionalmente decidido, pero su finalidad, base, plazo, acceso y riesgo de reidentificación continúan pendientes de aprobación Legal. | `DECISIÓN LEGAL` |
| `BRI-PLAN-001` | **Resuelto en `BRI-00C` (2026-09-19).** Existían tres planes locales activos no autorizados: `account-rule-compliance`, `crush-rule-compliance` y `crush-age-birthday` (este último con su carpeta maestra en Product vacía y una regla canónica citada inexistente). El usuario decidió: adoptar formalmente `account-rule-compliance` y `crush-rule-compliance` (añadidos a `AGENTS.md` §3 y a "Documentación permitida" de `api/AGENTS.md`); eliminar `crush-age-birthday` en ambos repos, trasladando sus hechos estables ya desplegados en producción (`age_as_of`/`zodiac_id`) a la sección "Antecedente" de `crush-rule-compliance/README.md`. **Adenda (2026-09-19, durante `API-CRU-RED-01`):** apareció un cuarto artefacto huérfano, `tests/Feature/Relationship/CrushGenderIntegrityTest.php`, sin carpeta de plan propia, que codifica el contrato opuesto a `RP-CRU-001` (afirma que `gender_id` es estructural). Bajo el mismo mandato, queda confirmado como obsoleto: `API-CRU-01` lo reemplaza, no lo conserva. | `CONTRADICCIÓN DOCUMENTAL` — resuelta; los correctivos de contenido de `account-rule-compliance`/`crush-rule-compliance` se autorizaron esa misma noche bajo mandato explícito del usuario (ver §8) |
| `BRI-ACC-001` | **Parcialmente resuelto (2026-09-19)**: `API-CRU-01` + `IOS-CRU-01` retiraron `gender_id` de alta, edición, recursos de respuesta y payloads iOS. El recontraste coordinador confirmó que `ListCrushesRequest` todavía lo acepta, `CrushController::applyListFiltersAndSorting()` filtra por `crushes.gender_id` y `CrushAttributeDistributionService::fromCrushes()` incorpora valores históricos a estadísticas. | `INCUMPLIDA` |
| `BRI-ACC-002` | **Resuelto en ambos extremos (2026-09-19)**: `API-CRU-01` (rechazo `prohibited` de `age_as_of`, sin excepción transitoria) + `IOS-CRU-01` (deja de calcularlo/enviarlo). | `CONFORME` |
| `BRI-ACC-003` | **Resuelto en `BRI-00C` (2026-09-19).** El usuario confirmó que el perfil de cuenta conserva `birth_date` completo (texto vigente de `RP-PER-002`/`RP-PER-010`); no se migra a `birth_year`-only. Los tres tests de `api/tests/Feature/Identity/AccountMinimalDataTest.php` que asumían `birth_year` (`registering_on_the_exact_eighteenth_birthday_exposes_only_the_birth_year_afterwards`, `registering_persists_only_the_birth_year_in_durable_state_never_day_or_month`, `profile_view_never_exposes_full_birth_date_exact_age_name_or_phone`) quedan confirmados como obsoletos; `account-rule-compliance` debe sustituirlos por cobertura de `birth_date` completo antes de `API-ACCOUNT-04`. No se toca código/tests desde esta sesión de coordinación. | `CONTRADICCIÓN DOCUMENTAL` — resuelta |

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
| Estadísticas | [Estadísticas](../../../rules/STATISTICS.md) |
| Tutoriales, formularios y accesibilidad | [Uso de la aplicación](../../../rules/UX.md) |
| Baja, inactividad, tareas automáticas y retención | [Ciclo de vida](../../../rules/DATA_LIFECYCLE.md) |
| Privacidad, seguridad y derechos | [Privacidad](../../../rules/PRIVACY.md) |

La matriz de Account y Crush ya está iniciada (ver más abajo); el resto de dominios sigue sin matriz. Los hallazgos previos de §5 se conservan como entradas de investigación.

## Matriz BRI-00B — Account y Crush (subalcance autorizado)

**Método y límites de esta pasada.** Evidencia obtenida mediante `codebase-memory-mcp` sobre índices frescos de `Users-santo-Sites-mdl-api` (generación `2026-09-19T10:49:22Z`) y `Users-santo-Sites-mdl-ios` (generación `2026-09-19T10:49:27Z`), ambos reindexados sobre los worktrees actuales. Se contrastaron directamente producción, consumidores y tests de las 26 filas de Account que seguían `NO DEMOSTRADA`. La migración de `federated_identities`, excluida por diseño del índice, y los tres rangos de una línea que el parser marcó parciales en `AuthViewModel.swift` se leyeron directamente. No se ejecutaron builds ni tests desde esta tarea coordinadora: una fila sólo pasa a `CONFORME` cuando el recorrido vigente coincide además con una prueba concreta incluida en las suites verdes ya registradas; cuando falta esa prueba específica queda `PARCIAL`.

**Aviso de WIP concurrente.** Ambos worktrees tienen cambios sin commitear de `relationship-redesign` (API: `CreateCrushUseCase`, `CreatePeriodUseCase`, `UpdatePeriodUseCase`, etc.; iOS: `CrushFormModel`, `EditCrushFormModel`, `CrushEditViewModel`, pasos de formulario de Crush). El grafo indexa ese WIP porque el watcher no distingue commit de working tree. Las filas de esta matriz que tocan alta/edición de Crush pueden moverse cuando ese WIP se cierre o se revierta; se marca `(WIP)` donde aplica.

**Planes locales adoptados.** `api/docs/plans/active/account-rule-compliance/README.md` y `api/docs/plans/active/crush-rule-compliance/README.md` ya contienen una auditoría RED por regla para buena parte de Account/Crush en API, con comandos de test ejecutados y registrados; ambos quedaron adoptados formalmente en `AGENTS.md` §3 y `api/AGENTS.md` "Documentación permitida" por decisión del usuario en `BRI-00C` (ver `BRI-PLAN-001`, resuelto). El tercer plan huérfano, `crush-age-birthday`, se eliminó; su contrato `age`/`age_as_of`/`zodiac_id` ya desplegado en producción quedó documentado como antecedente dentro de `crush-rule-compliance/README.md`. Esta matriz reutiliza la evidencia de los dos planes adoptados citándola, sin duplicar su texto ni asumir que sus fases de implementación ya están aceptadas.

### Account

**Resultado vigente de la auditoría completa de Account (77 filas):** `70 CONFORME`, `3 PARCIAL`, `3 INCUMPLIDA`, `1 NO APLICABLE`, `0 NO DEMOSTRADA`. Tras la recepción de `API-ACCOUNT-10`, las 26 filas pendientes de esta pasada quedaron resueltas como 24 conformes, 1 parcial y 1 incumplida.

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
| `BR-3.01` | API | `CONFORME` | Campos exigidos correctos en `RegisterRequest`; `PATCH /api/users` ya rechaza `email`/`password`/`plan_id` vía la allowlist de `API-ACCOUNT-05` (ver `RP-PER-004`, resuelto) |
| `BR-3.02` | API | `CONFORME` | `terms_version`/`privacy_version` validados contra `is_active` en `RegisterRequest::rules()` |
| `BR-3.03` | Ambas | `PARCIAL` | `normalizeEmail()` existe (`NormalizesTextData`) pero su aplicación uniforme en todas las superficies depende de `API-ACCOUNT-04b` (cifrado/lookup), excluida esta noche. Propietario: `user-information` |
| `BR-3.04` | API | `CONFORME` | `RegisterRequest::rules()`: `min:8` + 4 regex exactos, incluida la lista `@$!%*#?&` |
| `BR-3.05` | API | `CONFORME` | **Resuelto en `API-ACCOUNT-05` (2026-09-19).** `AuthController::tokenExpiresAt()` fija expiración explícita de 7 días en register/login/Apple; sin renovación por actividad |
| `RP-CUE-001` | Ambas | `CONFORME` | Ver `BRI-ACC-003` (resuelto): el contrato es `birth_date` completo. `RegisterUserUseCase` lo persiste; los 6 tests de `birth_year` (`AccountMinimalDataTest` x3, `AppleRegisterTest` x2, `UserTest` x1) ya fueron sustituidos por cobertura correcta (`API-ACCOUNT-04a`/`08`) |
| `RP-CUE-002` | API | `CONFORME` | **Resuelto en `API-ACCOUNT-05` (2026-09-19).** `LoginUseCase` dividido en `validate()` (sólo lectura) + `completeLogin()` (única mutación), invocado sólo tras `Auth::attempt()` exitoso — un password incorrecto ya no muta nada. Evidencia: `AuthTest` verde |
| `RP-CUE-003` | Ambas | `CONFORME` | `AuthViewModel.checkAuthentication()` no publica la sesión hasta resolver estado de email, perfil y activación Vault; cualquier token realmente inválido termina en `logout()`. `VaultSessionIntegrationTests` cubre restore pendiente, restore verificado en orden, activación Vault previa a publicar autenticación y token inválido sin perfil/Vault |
| `RP-CUE-004` | iOS | `CONFORME` | `AuthViewModel.logout()` revoca la sesión API y limpia JWT, identificador Apple, perfil, estado de verificación, password en memoria y contexto Vault mediante el disposer inyectado por `MDLApp`; no borra la cuenta ni la persistencia Vault remota/local namespaced. Cubierto por `testLogoutDisposesVaultBeforePublishingUnauthenticatedState` y `testLogoutPurgesMemoryAndPreservesNamespacedPersistence` |
| `RP-CUE-008` | Ambas | `CONFORME` | `password` no reversible (`'password' => 'hashed'` en `User::casts()`); ninguna Resource devuelve `password` (`$hidden`) |

#### Verificación obligatoria por email

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `BR-3.12` | `CONFORME` | **Resuelto en `API-ACCOUNT-06` (2026-09-19).** `EnsureEmailIsVerified`/`EnsureProvisionalAccountScope` ahora incondicionales, sin bypass por flag — compatibilidad histórica retirada |
| `RP-VER-001` | `CONFORME` | `EmailVerificationFlowTest::send_code_creates_an_otp_and_emails_it`; `ensureVerificationCode()` no bloquea login ante fallo de envío (`AuthController::login()`) |
| `RP-VER-002` | `CONFORME` | **Resuelto en `API-ACCOUNT-10` (2026-09-19).** `createAndSendCode()` ahora comprueba `locked_until` antes de `checkCooldown()`, sobre la misma fila bajo `lockForUpdate()`; devuelve el mismo contrato `429`/`retryAfter` sin tocar la fila ni enviar mail. `EmailVerificationFlowTest::a_resend_requested_while_locked_out_does_not_replace_the_locked_otp` verde |
| `BR-3.13` | `CONFORME` | `EMAIL_VERIFICATION_PROVISIONAL_ACCOUNT_DAYS`; test `ProvisionalAccountDeadlineTest::registration_sets_a_fixed_seven_day_provisional_deadline` y `resending_the_verification_code_does_not_change_the_deadline`, ambos verdes |
| `BR-3.14` | `CONFORME` | Resuelto junto con `BR-3.12` |
| `RP-VER-003` | `CONFORME` | Resuelto junto con `BR-3.12` |
| `BR-3.15` | `CONFORME` | `ProcessProvisionalAccountExpirationsCommandTest`: baja íntegra, libera email, sin token/Vault/lead (heredado, verde) |
| `RP-VER-005` | `CONFORME` | `AppleRegisterTest::it_creates_a_verified_apple_account_and_returns_a_session` y `no_otp_is_sent_and_no_verification_row_is_created_for_a_verified_apple_signup`, verdes |

#### Acceso con Apple — alta y vinculación

Autoridad: `AppleAuthController`, `RegisterAppleUserUseCase`, `FederatedAuthContinuationService`, `AppleRegisterRequest`. iOS: `AuthViewModelAppleTests`, `AppleSignInViewModelTests`, `AppleLinkView`.

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-APL-001` | `CONFORME` | `AppleAuthController::register()`/continuación; cubierto por `AppleRegisterTest`/`AppleAuthorizeTest` |
| `RP-APL-002` | `CONFORME` | Flujo de autorización Apple obligatorio antes de continuar (`AppleAuthorizeTest`) |
| `RP-APL-003` | `CONFORME` | La migración impone unicidad de `(provider, subject_digest)` y `(user_id, provider)`; register/link vuelven a comprobar ambos lados y traducen carreras de constraint a conflicto completo. `AppleRegisterTest`, `AppleLinkTest` y `AppleLinkFromSessionTest` cubren ambos conflictos |
| `RP-APL-004` | `CONFORME` | `AppleSignInViewModelTests` cubre fallos de autorización sin crear sesión |
| `RP-APL-005` | `CONFORME` | `AppleRegisterRequest::rules()`: `birth_date` obligatorio, mismo cálculo de 18 años que `RegisterRequest`; sin `password` |
| `RP-APL-006` | `CONFORME` | API toma `email` e `is_private_email` exclusivamente de los claims Apple verificados y conserva ambos en la identidad federada; alta usa ese email, sea personal o relay. `AppleRegisterTest`/`AppleLinkTest` cubren alta y Hide My Email |
| `RP-APL-008` | `CONFORME` | `AuthorizeAppleIdentityUseCase` no emite continuación sin email presente y verificado; iOS no envía un email alternativo en authorize/register. Tests: `a_missing_email_claim_fails_closed_without_a_continuation` y `an_unverified_email_claim_fails_closed_without_a_continuation` |
| `RP-APL-009` | `CONFORME` | iOS solicita sólo scope `.email`, los DTO/use cases de API no tienen nombre Apple y `AppleRegisterTest::a_name_claim_supplied_by_apple_is_never_persisted` lo demuestra |
| `RP-APL-010` | `CONFORME` | `AuthorizeAppleIdentityUseCase` emite la continuación con TTL de 600 s y register/link la consumen antes de operar, por lo que es single-use incluso ante fallo. Tests de continuación vencida, desconocida y reutilizada en `AppleRegisterTest`/`AppleLinkTest` |
| `RP-APL-011` | `CONFORME` | `RegisterAppleUserUseCase` encierra User, aceptaciones, identidad federada y emisión de sesión en una única transacción; los conflictos concurrentes/identidad ya usada prueban rollback completo |
| `RP-APL-012` | `CONFORME` | Cancelar la hoja nativa devuelve `AppleSignInViewModel` a `.idle`, no llama API ni muestra error; el formulario todavía no existe en ese punto, por lo que no hay datos introducidos que perder. Cubierto por `testHandleAppleCompletionCancellationReturnsToIdleWithoutAnyErrorMessage` |
| `RP-APL-014` | `INCUMPLIDA` | `AppleRegisterFlowView` pasa directamente por nacimiento y aceptación legal; ni la vista ni las claves localizadas muestran antes de confirmar que se usará el email entregado por Apple. `IOS-ACCOUNT-01` añadió el test RED sobre la vista real y su control de harness; implementación aceptada con validación pendiente porque el WIP de `relationship-redesign` no compila. `IOS-ACCOUNT-02` sigue bloqueada |
| `RP-APL-024` | `CONFORME` | El relay se guarda como email vigente verificado sin exigir dirección personal; recuperación opera sobre cualquier email de la cuenta y `AppleLinkTest::linking_works_when_the_apple_email_differs_from_the_mdl_email_hide_my_email` conserva ese contrato. El estado de forwarding se actualiza sin sustituir el relay |
| `RP-APL-017` | `CONFORME` | `UserTest::a_password_account_reports_password_as_its_only_auth_method` y simétrico Apple; ambos métodos conviven en `User` |
| `RP-APL-018` | `CONFORME` | **Resuelto en `API-ACCOUNT-07`.** `POST /auth/apple/link-from-session` + `LinkAppleFromSessionUseCase`, mismas garantías 1:1/single-use que el flujo público pero con el `User` de la sesión Sanctum |
| `RP-APL-015` | `CONFORME` | **Resuelto de punta a punta (2026-09-19).** API: `POST /auth/apple/credential/request`+`/confirm`, alta atómica con revocación de sesiones. iOS: `AppleCredentialView`/`AppleCredentialViewModel` (pantalla nueva en Settings, gateada por `currentUserHasPassword == false`), fuerza `logout()` tras el éxito |
| `RP-APL-016` | `CONFORME` | Mismo flujo que `RP-APL-015` |
| `RP-APL-026` | `CONFORME` | Resuelto junto con `RP-APL-015`: cambiar/verificar email por el flujo general no crea contraseña; el alta de credencial es la acción distinta confirmada por separado |
| `RP-APL-032` | `CONFORME` | Mismo flujo que `RP-APL-015`; revocación de sesiones ya atómica (`RP-OTP-007`) |
| `RP-APL-027` | `CONFORME` | Un email Apple que coincide con una cuenta existente produce `linkRequired`, nunca sesión automática; `AppleLinkView` exige email/password y confirmación. Cubierto también con casing distinto en `AppleAuthorizeTest` (ese test permanece RED sólo por `BR-3.03`, no por bypass de vínculo) |
| `RP-APL-028` | `CONFORME` | Todo `registrationRequired`, incluido relay/email distinto, presenta `AppleRegisterFlowView` con la acción «Ya tengo una cuenta MDL», que reutiliza la misma continuación y cambia al vínculo explícito |
| `RP-APL-029` | `CONFORME` | El vínculo público combina autorización Apple ya contenida en la continuación, email/password MDL y submit explícito. Si no se conoce la contraseña, se cancela la continuación, se usa la recuperación pública y se inicia un vínculo nuevo; API no ofrece un atajo que omita esa acreditación |
| `RP-APL-030` | `CONFORME` | **Resuelto de punta a punta (2026-09-19).** API: `POST /auth/apple/link-from-session`, `AppleLinkFromSessionTest` 11/11. iOS: `AppleLinkFromSessionView`/`AppleLinkFromSessionViewModel` (confirmación explícita antes de autorizar, nunca adopta sesión de otra cuenta ante conflicto `RP-APL-033`, no revoca sesiones) |
| `RP-APL-031` | `CONFORME` | `LinkAppleUserUseCase` nunca modifica `email` ni `email_verified_at`; una cuenta no verificada conserva el mismo gate de API/iOS aunque obtenga un método Apple. `AppleLinkTest::linking_preserves_the_existing_account_email_and_verification_state` lo cubre |
| `RP-APL-033` | `CONFORME` | **Resuelto en `API-ACCOUNT-07` (2026-09-19).** `AppleLinkFromSessionTest`: identidad ya vinculada a otra cuenta rechaza completo, sin fusionar/absorber/transferir |
| `RP-APL-020` | `CONFORME` | Notificaciones Apple verificadas `consent-revoked`/`account-deleted` revocan la identidad e invalidan tokens dentro de la transacción sin borrar User/diario; iOS también cierra localmente ante credential state `.revoked`/`.notFound`. Cubierto por `AppleNotificationsEndpointTest` y `AuthViewModelAppleLifecycleTests` |
| `RP-APL-023` | `CONFORME` (iOS) | `AppleAuthorizationErrorMapper.map` + manejo en `AuthViewModelAppleTests`/`AppleSignInViewModelTests` para fallos de disponibilidad |

#### Perfil de la cuenta de usuario

Autoridad: `UpdateUserRequest`, `UserController::update()`, `UserResource::toArray()`.

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-PER-001` | `CONFORME` | Omitir conserva; allowlist estricta (`API-ACCOUNT-05`) cubre el resto. Ningún campo actual del perfil admite borrado explícito, consistente con la regla |
| `RP-PER-002` | `CONFORME` | Ver `BRI-ACC-003` (resuelto). `UserResource::toArray()` expone `birth_date` completo, coincidente con la regla; los tests de `birth_year` en `AccountMinimalDataTest.php` quedan pendientes de sustitución, no la implementación |
| `RP-PER-003` | `PARCIAL` | Implementación conforme: Settings sólo ofrece `en`/`es`, el cambio actualiza `Bundle`/locale y `AppleLanguages`, y `LocalizationManager` restaura la preferencia al iniciar. `IOS-ACCOUNT-01` añadió el recorrido UI específico que selecciona idioma, termina y relanza sin limpiar almacenamiento/sesión; su implementación fue aceptada, pero todavía no pudo ejecutarse por el bloqueo de compilación ajeno de `relationship-redesign` |
| `RP-PER-004` | `CONFORME` | **Resuelto en `API-ACCOUNT-05` (2026-09-19).** `UpdateUserRequest::withValidator()` implementa allowlist estricta (`birth_date`/`gender_id`/`country_id` únicos) vía hook `after()`: cualquier otra clave, incluida `email`/`password`/`plan_id`/`phone`, rechaza la petición completa con 422 |
| `RP-PER-007` | `CONFORME` | `UserResource::toArray()` devuelve `gender_id`/`country_id`; `AccountMinimalDataTest::profile_view_never_exposes_full_birth_date_exact_age_name_or_phone` confirma que siguen presentes como catálogo |
| `RP-PER-008` | `CONFORME` | Nombre/edad ya excluidos del contrato HTTP; **`phone` ahora también removido de `UserResource`** (`API-ACCOUNT-04a`, 2026-09-19) — ya no se expone en ninguna respuesta. La columna/mutator físicos siguen vivos a nivel de modelo (retiro físico pendiente, propietario `user-information`), pero no hay ninguna superficie HTTP que los use |
| `RP-PER-009` | `CONFORME` | Resuelto junto con `RP-PER-004`: allowlist estricta vía `UpdateUserRequest::withValidator()` |
| `RP-PER-010` | `CONFORME` | `UserResource::toArray()` expone `birth_date`, `gender_id`, `country_id`; no expone edad exacta calculada |

#### Protección del email, cambio de email y contraseña

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-VLT-005` | `INCUMPLIDA` | Confirmado leyendo `app/Models/User.php`: `email` no tiene cast de cifrado ni mutator; `search_code` no encuentra `email_lookup` en `app/`. Coincide con `ARCHITECTURE.md` §10.1 ("cifrado de campo y búsqueda pendientes en `user-information`"). Propietario: `user-information` (API-UI-00/01), no BRI |
| `RP-CUE-009` | `INCUMPLIDA` | Mismo hallazgo: sin cifrado de campo, ninguna de las copias enumeradas está protegida hoy. Propietario: `user-information` |
| `RP-MAIL-001` | `CONFORME` | Las rutas de cambio están autenticadas y tras `ensure_email_verified`; `requestEmailChange()` vuelve a comprobar que el email actual esté verificado. Test: `change_request_is_rejected_when_current_email_is_not_verified` |
| `RP-MAIL-002` | `CONFORME` | La solicitud crea/envía OTP para la dirección nueva y no muta `users.email`; iOS sólo pasa a confirmación. Test: `change_request_sends_the_code_to_the_new_email_and_keeps_the_old_one_active` |
| `RP-MAIL-003` | `CONFORME` | Confirmación bloquea User+OTP, valida y revalida unicidad antes de actualizar email/verificación; código erróneo o colisión concurrente preservan el anterior. Tests de éxito, código erróneo y carrera en `EmailVerificationFlowTest` |
| `RP-MAIL-004` | `CONFORME` | **Resuelto en `API-ACCOUNT-09` (2026-09-19).** `EmailVerificationFlowTest::changing_the_email_preserves_the_linked_apple_identity`: cambia el email por los endpoints reales y confirma `provider`/`subject`/`subject_digest`/`status` intactos, y que la misma identidad sigue resolviendo la misma cuenta vía `FederatedIdentityRepositoryInterface::findByProviderAndSubjectDigest()` (el binding real de producción). Verde a la primera corrida |
| `RP-MAIL-005` | `CONFORME` | **Resuelto en `API-ACCOUNT-07` (2026-09-19).** `AppleAddCredentialTest`: email ya usado por otra cuenta rechaza completo, sin fusionar/absorber/transferir |
| `RP-PWD-001` | `CONFORME` | **Resuelto de punta a punta (2026-09-19).** API: `POST /auth/password/change` + `ChangePasswordRequest`. iOS: `ChangePasswordView`/`ChangePasswordViewModel` (pantalla nueva en Settings, gateada por `currentUserHasPassword == true`), fuerza `logout()` tras el éxito |
| `RP-PWD-002` | `CONFORME` | `ChangePasswordTest`: actual incorrecta/confirmación distinta/nueva inválida rechazan completo sin mutar |
| `RP-PWD-003` | `CONFORME` | **Resuelto en `API-ACCOUNT-06/07`.** Cambio exitoso preserva email/verificación/Apple/id/Vault/diario y revoca todas las sesiones atómicamente |
| `RP-PWD-004` | `CONFORME` | Una cuenta Apple sin contraseña usa exclusivamente el flujo de `RP-APL-015` (`credential/request`+`confirm`), nunca `ChangePasswordRequest` |
| `BR-3.06` | `CONFORME` | **Resuelto en `API-ACCOUNT-06` (2026-09-19).** Cooldown anti-enumeración movido a `Cache`, evaluado antes de mirar si la cuenta existe; nunca crea fila para email inexistente. Evidencia: `PasswordRecoveryContractTest` |
| `RP-OTP-001` | `CONFORME` | **Resuelto en `API-ACCOUNT-06`.** `verifyOtp()`/`resetPassword()` con `lockForUpdate()`+transacción; confirmado contra MySQL/InnoDB real (`ConcurrentPasswordResetTest`) |
| `BR-3.07` | `CONFORME` | `PasswordResetService` conserva estado anti-enumeración en cache y aplica los tres escalones 60/300/3600 s; `PasswordRecoveryContractTest::forgot_password_resend_cooldown_escalates_through_three_tiers` está verde |
| `BR-3.08` | `CONFORME` | **Resuelto en `API-ACCOUNT-10` (2026-09-19).** `PasswordResetService::sendOtp()` ahora decide dentro de un `DB::transaction()` con `lockForUpdate()` sobre la fila más reciente; si está bloqueada, no muta nada y devuelve el mismo `sent` genérico sin llegar nunca a `Mail::send()` (`BR-3.06` intacto). Carrera real demostrada contra MySQL/InnoDB (`ConcurrentPasswordOtpLockoutTest`, nuevo). `PasswordRecoveryContractTest::a_forgot_password_call_while_locked_out_does_not_replace_the_recovery_otp` verde |
| `BR-3.16` | `CONFORME` | Recovery responde `invalid_or_expired_code` para incorrecto, vencido y bloqueado tanto al verificar como al resetear; `PasswordResetFlowFullTest` confirma que nunca filtra `account_locked` ni estado interno |
| `RP-OTP-005` | `CONFORME` | `resetPassword()` sólo actualiza el verificador, revoca sesiones y consume OTP; no lee, reencapsula ni borra Vault. `PasswordResetSessionRevocationTest::a_successful_reset_never_touches_the_vault` está verde |
| `RP-OTP-007` | `CONFORME` | **Resuelto en `API-ACCOUNT-06` (2026-09-19).** Revocación atómica de todas las sesiones al establecer/cambiar/recuperar contraseña |

**Bloque implementado; recepción coordinadora con validaciones pendientes (2026-09-19), fases `API-ACCOUNT-04a/05/06/07/08/09/10` e implementación de tests de `IOS-ACCOUNT-01`.** `RP-CUE-002`, `BR-3.05`, `RP-PER-001/004/008/009`, `RP-OTP-001/007`, `RP-PWD-001..004`, `RP-APL-015/016/018/026/030/032/033`, `RP-MAIL-004/005`, `RP-VER-002/003`, `BR-3.06`, `BR-3.08` y `BR-3.12..15` cuentan con implementación o evidencia directa y tests en el worktree — ver filas individuales. La recepción confirmó las rutas y símbolos principales, pero no reejecutó las suites reportadas por Claude. `API-ACCOUNT-10` dejó verdes los dos locks de regresión de `API-ACCOUNT-09` y demostró la nueva frontera transaccional de recovery contra MySQL/InnoDB real, preservando la respuesta genérica. En iOS, `IOS-ACCOUNT-01` tiene implementación aceptada pero no fase aceptada por el bloqueo de compilación ajeno; `IOS-ACCOUNT-02` sigue bloqueada. `API-ACCOUNT-04b` continúa pendiente en `user-information`. Account todavía no puede cerrarse.

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
| `RP-CRU-013` | `INCUMPLIDA` | Alta, edición, respuestas e iOS ya respetan el inventario, pero la API todavía interpreta `gender_id` en filtros de listas y estadísticas. Ver `RP-CRU-001`; la columna histórica puede permanecer físicamente sólo si queda completamente inerte. |
| `RP-CRU-014` | `CONFORME` | `height_id`/`eye_color_id`/`hair_color_id`/`body_type_id`/`skin_color_id` viajan como enteros de catálogo en `UpdateCrushRequest`/`CrushResource`, nunca texto libre |
| `RP-CRU-015` | `INCUMPLIDA` | El inventario funcional cerrado todavía se amplía de hecho mediante dos consumidores de `gender_id` legible por la API: filtrado de listas y distribución estadística. |
| `BR-4.01` | `INCUMPLIDA` | Género y `age_as_of` ya están corregidos en API/iOS, pero el texto vigente exige una experiencia inicial completa. API conserva `first_experience` como `sometimes`; iOS considera válido el paso cuando los tres campos están vacíos; Android conserva el mismo bloque opcional y tests que exigen ese comportamiento. |
| `RP-CRU-004` | `INCUMPLIDA` | `CreateCrushUseCase` sí usa una transacción para las partes presentes, pero permite crear Crush + período sin experiencia. iOS y Android sólo crean la entry/parte de Experience cuando el bloque opcional está completo. El contrato vigente exige siempre las tres partes o ninguna. |
| `RP-CRU-005` | `CONFORME` | **Resuelto en `API-CRU-01` (2026-09-19).** `PostCrushRequest`/`UpdateCrushRequest` ahora restringen `ducky_id` con `Rule::in([Duck::CASUAL_DUCK, ..., Duck::UGLY_DUCK])` en alta y edición, evidencia ejecutada (`CrushProfileContractTest`, 0 regresión). iOS ya era `CONFORME` (`enum CrushDucky`). |

#### Perfil — contenido exclusivo del Vault

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-CRU-001` | `INCUMPLIDA` | **Parcialmente corregido (2026-09-19).** Alta, edición, respuestas, mapa y payloads iOS ya no admiten/devuelven `gender_id`; el género sigue dentro del payload Vault cifrado. Sin embargo, `ListCrushesRequest::rules()` aún lo acepta, `CrushController::applyListFiltersAndSorting()` consulta `crushes.gender_id`, `EloquentCrushRepository` hidrata/persiste el valor histórico y `CrushAttributeDistributionService::fromCrushes()` lo agrega a `genderDistribution`. Esos caminos hacen que la API todavía interprete género fuera del Vault. |
| `RP-CRU-012` | `CONFORME` | `profession`/`whatsapp_*`/redes/`birthdate` no tienen columna ni contrato en la API funcional; `CrushViewModel` los superpone exclusivamente desde el payload Vault descifrado. `CrushListViewModel.performSearch()` exige Vault desbloqueado y compara localmente nombre/país; `VaultReadIntegrationV2Tests` demuestra que el término nunca viaja a la API. |
| `RP-CRU-023` | `INCUMPLIDA` (iOS) | Auditoría fresca (2026-09-19): redes y WhatsApp sí aplican 255/20 en `Step3_SocialsView`, incluida limpieza conjunta del prefijo al quitar el número; sin embargo `CrushInfoFormContent.nameField` admite 255 en vez de 191 y `Step2_BirthdayView.professionField` usa un `TextField` sin límite. La regla completa no se cumple. |

#### Rasgos físicos estructurados

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-CRU-016` | `CONFORME` | Cinco campos opcionales independientes, enteros de catálogo, confirmados en `UpdateCrushRequest::rules()` |
| `RP-CRU-017` | `CONFORME` | Cada selector enlaza una propiedad independiente de `EditCrushFormModel`; `toAPIRequest()` emite sólo el rasgo informado/cambiado y `UpdateCrushUseCase` aplica cada campo opcional sin tocar el resto. `UpdatePayloadTests::testCrushEdit_OmitUnchangedFields` y `CrushProfileContractTest::the_five_catalogued_physical_traits_are_still_accepted_persisted_and_returned_via_update` cubren el contrato. |
| `RP-CRU-018` | `CONFORME` | `ColorPickerGridView.toggleSelection` e `IconPickerGridView` convierten un segundo toque sobre la opción seleccionada en `nil`; `CrushEditViewModel.buildPayload` lo serializa como `NSNull` y API lo procesa por `explicitNulls`/`CrushEntity::clearFields()` sólo para el campo nombrado. Evidencia: `UpdatePayloadTests::testCrushEdit_SendNullForClearedFields`. |

#### Lectura, búsqueda y edición del perfil

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-CRU-019` | `CONFORME` (iOS) | `VaultUIManagerTests`, `AppLock*Tests` cubren bloqueo del Vault al iniciar/volver de background sobre listas/mapa/Timeline/fichas |
| `RP-CRU-020` | `PARCIAL` | La ruta de producción sí combina ambos orígenes: `CrushViewModel.performLoadCrushViewData()` carga estructura API y superpone desde `CrushVaultJSONResolver` nombre, profesión, WhatsApp, redes, género y cumpleaños; `CrushProfileView` presenta cabecera, `SocialMediaCard`, `ProfileDataCard`, `CharacteristicsCard` y `RatingsCard`, sin IDs/timestamps como datos de perfil. Hay pruebas de resolución Vault y decodificación API sin PII (`CrushViewModelBirthdayTests`, `PrivateFieldContractCutTests`), pero no una prueba de presentación que recorra conjuntamente todos los campos del inventario; queda evidencia automatizada específica pendiente. |
| `RP-CRU-021` | `CONFORME` | `CrushListViewModel.performSearch()` sólo arranca con Vault desbloqueado; `performLocalSearch()` obtiene IDs por nombre desde entradas locales, normaliza nombre/país con `folding(.diacriticInsensitive, .caseInsensitive)` y envía a API únicamente página, límite y filtros estructurales. `VaultReadIntegrationV2Tests::testPerformSearchFindsMatchOnLaterPageRespectsStructuralFiltersAndNeverLeaksTermToAPI` y los tests de nombre/país cubren coincidencia parcial, mayúsculas, acentos y ausencia del término en red. |
| `RP-CRU-022` | `CONFORME` | Resuelto en ambos extremos (`API-CRU-01` + `IOS-CRU-01`, 2026-09-19) — ver `RP-CRU-001`. El triestado de `zodiac_id` sigue confirmado (`UpdateCrushCommand::hasZodiacId` + `CrushEntity::updateZodiac`). |

#### Fecha de encuentro

Autoridad: `UpdateCrushUseCase::handleMetAtUpdate()`/`validateChronologicalIntegrity()`, regla `CrushMetAtBeforeEntities`.

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-CRU-007` | `PARCIAL` | `PostCrushRequest`: `met_at` con `before_or_equal:today` (sin fecha futura, correcto). El límite frente al primer acontecimiento independiente es el hallazgo de `RP-CRU-008` |
| `RP-CRU-008` | `INCUMPLIDA` | `validateChronologicalIntegrity()` acepta igualdad porque sólo rechaza una experiencia posterior con fecha `< newMetAt`; el contrato exige que `met_at` sea estrictamente anterior. Además sigue tratando los períodos posteriores como límite independiente, aunque el texto vigente deriva el límite exclusivamente de la primera experiencia posterior. Propietario: `API-CRU-02` (`crush-rule-compliance`). |
| `RP-CRU-009` | `INCUMPLIDA` | `handleMetAtUpdate()` guarda Crush, período y experiencia mediante tres escrituras separadas, sin transacción/lock, y no desplaza el hito inicial. Un fallo intermedio deja el bundle desincronizado. Propietario: `API-CRU-02`. |

#### Valoraciones

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `BR-4.02` | `CONFORME` | **Resuelto en `API-CRU-01` (2026-09-19).** `UpdateCrushRequest` ahora exige `integer\|between:1,5` en las cuatro valoraciones y `rate_global` es `prohibited`. El bug de borrado explícito (`isset()` no distinguía `null` de omitido) también se corrigió: las cuatro valoraciones se movieron al mismo mecanismo `explicitNulls`/`clearFields()` que los rasgos físicos, y `CrushEntity::clearFields()` recalcula `rate_global` al limpiar una. Evidencia: `CrushRatingContractTest` verde, 0 regresión. |

#### Edad, cumpleaños y zodíaco

Autoridad: `UpdateCrushCommand`/`CrushEntity` (triestado); antecedente de despliegue documentado en `api/docs/plans/active/crush-rule-compliance/README.md` (ex `API-AGE-01/02/C1`, ya desplegadas en producción).

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-AGE-001` | `CONFORME` | `age` validado `between:18,80` en alta y edición |
| `RP-AGE-002` | `CONFORME` | **Resuelto en `API-CRU-01` (2026-09-19).** `age_as_of` es `prohibited` en `PostCrushRequest`/`UpdateCrushRequest` (alta y edición); la API lo genera siempre server-side con la fecha vigente. Evidencia: `CrushAgeAsOfClientRejectionTest` verde. |
| `RP-AGE-003` | `CONFORME` | `PartialBirthday` sólo contiene `day`/`month` y serializa el formato Vault canónico `--MM-dd`; su parser rechaza cualquier fecha completa u otro formato. `PartialBirthdayTests::testLocalizedDisplayNeverIncludesYear` y los round-trips de `CrushEditViewModelVaultTests` cubren persistencia/presentación sin año. |
| `RP-AGE-004` | `CONFORME` | `PartialBirthday.Zodiac.forBirthday(day:month:)` deriva uno de los doce signos desde una fecha real; `EditCrushFormModel.ageAndZodiacParameters()` envía sólo `zodiac_id`, nunca día/mes/cumpleaños. `PartialBirthdayTests` cubre los doce signos y fechas imposibles; `EditCrushFormModelBirthdayTests::testValidBirthdayChangeSendsCorrectZodiacId` cubre el payload. |
| `RP-AGE-005` | `CONFORME` | `AgeCalculator.visibleAge/presentedAge` no aceptan `metAt`; el zodiaco tampoco lo usa. `EditCrushFormModelBirthdayTests::testMetAtNeverAffectsZodiacCalculation` fija la regresión. |
| `RP-AGE-006` | `CONFORME` | `AgeCalculator.presentedAge()` suma únicamente los cumpleaños posteriores a `age_as_of` y hasta hoy cuando el cumpleaños está disponible. `AgeCalculatorTests::testPresentedAgeWithValidBirthdayAndAnchorIsExact` y los tests de cruce antes/después del cumpleaños lo cubren. |
| `RP-AGE-007` | `CONFORME` | Sin cumpleaños, `presentedAge()` devuelve edad exacta el propio día del ancla y después calcula el rango mínimo/máximo sobre todos los día/mes válidos. Cubierto por `testPresentedAgeWithoutBirthdaySameDayAsAnchorIsExact`, `...AfterAnchorIsRangeNeverBelowSnapshot` y `...MoreThanAYearAfterAnchorShiftsRangeUpward`. |
| `RP-AGE-008` | `CONFORME` | El mínimo del intervalo parte de `age + minimumCrossed`, donde `minimumCrossed` nunca es negativo; `testPresentedAgeWithoutBirthdayAfterAnchorIsRangeNeverBelowSnapshot` lo afirma explícitamente. |
| `RP-AGE-009` | `CONFORME` | `civilBirthdayDate()` convierte 29/2 en 28/2 sólo en años no bisiestos. Evidencia: `testFeb29InNonLeapYearIsTreatedAsFeb28`, `testPresentedAgeWithoutBirthdayRangeIncludesFeb29TreatedAsFeb28InNonLeapYear` y `testPresentedAgeFeb29AndFeb28CrossSameCountInNonLeapYear`. |
| `RP-AGE-010` | `CONFORME` | `EditCrushFormModel.decideAgeSave()` devuelve `.omitPair` cuando edad/cumpleaños no cambian y `ageAndZodiacParameters()` omite la pareja; profesión, redes, rasgos y ratings no reanclan edad. Evidencia: `testUnrelatedFieldChangeOmitsAgePairAndZodiac` y su variante de snapshot legacy. |
| `RP-AGE-011` | `CONFORME` | **Resuelto en `API-CRU-01` (2026-09-19).** Ídem `RP-AGE-002`. El bug adicional de edición (`transient_age_update` descartaba un `age` sin pareja) también se corrigió: un `PATCH` con sólo `age` ahora lo actualiza y genera un `age_as_of` nuevo, igual que el alta. `CrushAgeContractTest::test_patch_transient_only_age_does_not_mutate_pair` se reemplazó por el test del contrato correcto. |
| `RP-AGE-012` | `CONFORME` | Al añadir/cambiar cumpleaños sin edición directa de edad, `decideAgeSave()` calcula la edad vigente desde `originalAge`/`originalAgeAsOf`, reancla en la fecha de confirmación y persiste el nuevo cumpleaños Vault. Evidencia: `testAddingBirthdayRecalculatesFromOriginalSnapshotAndReanchorsToday` y `testChangingBirthdayIgnoresDraftAgeAndUsesOriginalSnapshot`. |
| `RP-AGE-013` | `CONFORME` | `.clearedExplicitly` recalcula primero con el cumpleaños anterior, reancla la edad y envía `zodiac_id = null`; el snapshot Vault elimina día/mes. Evidencia: `testClearingBirthdayRecalculatesWithPreviousBirthdayAndSendsRealNSNull`, `testClearingBothFieldsFromExistingBirthdaySendsNullZodiacId` y `CrushEditViewModelVaultTests::testBirthdayClearsProperly`. |
| `RP-AGE-014` | `CONFORME` | La primera rama de `decideAgeSave()` da prioridad incondicional a `ageWasEdited` y reancla la edad introducida incluso si también cambió el cumpleaños. Evidencia: `testDirectAgeEditWinsOverBirthdayRecalculationWhenBothChange`. |
| `RP-AGE-015` | `PARCIAL` | Coherente con el contrato de pareja documentado como antecedente en `crush-rule-compliance` (`PATCH sin edad conserva pareja`, `PATCH con pareja actualiza ambos`), pero no hay todavía una ejecución de suite fresca registrada (ex `API-AGE-C1`: "no conserva aún un resultado literal de test run") |

#### Fotos del Crush (iOS)

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-FOT-001` | `CONFORME` | `LocalMediaManager`, sin subida a Vault/servidor para fotos de Crush en la beta |
| `RP-FOT-002` | `CONFORME` | `EXIFStripper.stripEXIF()`/`verifyNoEXIF()` con tests dedicados |
| `RP-FOT-003` | `CONFORME` | `LocalMediaManagerTests::testSavedPhotoFileIsExcludedFromBackup`, `testPerCrushDirectoryIsExcludedFromBackup`, `testRootLocalMediaDirectoryIsExcludedFromBackup` |
| `RP-FOT-004` | `CONFORME` | `LocalMediaManagerTests::testDeletingASinglePhotoStillRemovesOnlyThatFile` |
| `RP-FOT-005` | `CONFORME` | `LocalMediaManagerTests::testDeleteAllMediaStillRemovesTheWholeCrushDirectory`; baja de cuenta cubierta por `DependencyContainerAccountDeletionCleanupTests::testDirect2xxWipesSQLiteKeychainLocalMediaAndOnboardingState` |
| `RP-FOT-006` | `PARCIAL` | Las fotos activas se escriben como `.jpg` exclusivamente bajo `LocalMedia/{crushId}` mediante `MediaUploadService.uploadPhotos()` → `LocalMediaManager.saveImage()`; el flujo de recuperación Vault no referencia ni restaura `LocalMedia`. No existe borrado remoto garantizable para estas fotos locales y el código Vault-media remoto permanece sin caller de pantalla. `LocalMediaManagerTests` y `VaultSessionIntegrationTests::testLateMediaOperationFromADoesNotWriteAfterLoginB` cubren almacenamiento local y aislamiento, pero falta una prueba específica que ejecute recovery y confirme que las fotos no se restauran. |
| `RP-FOT-007` | `CONFORME` | **Resuelto (2026-09-19), decisión del usuario: bloquear selección al hueco restante.** `LocalMediaManager.maxPhotosPerCrush = 5` como fuente única; contador "X/5 fotos" antes de la galería (`CrushGalleryView.photoQuotaText`); botón de subir deshabilitado en 5 (`CrushViewModel.canAddMorePhotos`); `PHPickerConfiguration.selectionLimit` capado al hueco real (nunca `0`, que en PHPicker significa sin límite); segunda barrera pura (`itemsWithinPhotoQuota`) recorta y avisa ante una carrera. Evidencia: `CrushGalleryViewTests`, `CrushViewModelPhotoQuotaTests`, build+tests reales, 0 regresión |
| `RP-FOT-009` | `CONFORME` (regla nueva, 2026-09-19) | `PhotoPickerCrush.makeUIViewController` ya fija `config.filter = .images` — es el único picker vivo del perfil de Crush (confirmado sin otros callers de `PHPickerViewController`/`UIImagePickerController` en `Features/Crush/`). **Hallazgo colateral:** existe un segundo picker, `VaultPhotoPicker` (`MDLApp/Features/Vault/UI/Media/`), que sí admite `.any(of: [.images, .videos])`, junto con `VaultMediaGalleryView`/`VaultMediaGalleryViewModel`/`VaultMediaSyncManager` (este último con manejo completo de video: `EXIFStripper.stripEXIF(from: videoURL)`, `validateVideoLimits`). Los cuatro símbolos tienen **cero callers** en todo el grafo — no alcanzable desde ninguna pantalla activa, así que no contradice `RP-FOT-009` en tiempo de ejecución. **Decisión del usuario (2026-09-19): se conserva.** No es compatibilidad histórica a retirar — es código válido, presumiblemente el arranque de una feature futura (backup de medios al Vault, con soporte de video) todavía no conectada a ninguna pantalla. No tocar sin autorización explícita nueva. |
| `RP-FOT-008` | `CONFORME` | **Resuelto (2026-09-19):** el único caller de producción (`MediaUploadService.uploadPhotos`) llamaba con `shouldOptimize: false`, que saltaba por completo el único chequeo de tamaño (`handleLargeImage`/Tier 3) — el tope de 10 MB era código muerto, ninguna foto se rechazaba nunca. Corregido en `ImageConversionHelper.prepareImageForUpload()`: los tres caminos (sanitización/Tier 1/Tier 2-3) ahora pasan por un único `guard` final contra `absoluteMaxSizeLimitBytes`. Evidencia: `ImageConversionHelperTests` (5 tests nuevos, deterministas) + regresión en `LocalMediaManagerTests`/`VaultSessionIntegrationTests`, todo verde con `xcodebuild` real |

#### Mapa de Crushes por país (iOS)

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-MAP-001` | `CONFORME` | Rutas `GET /api/map`/`GET /api/map/{countryId}`, sólo lectura |
| `RP-MAP-002` | `CONFORME` | **Incumplimiento real encontrado y corregido (2026-09-19):** `DatabaseQueries::map()`/`mapCountryDetails()` (Query Builder crudo, sin scope de Eloquent) filtraban `is_anonymized`/`deleted_at` pero no `is_archived` — un Crush archivado seguía en el mapa. Corregido agregando `where('is_archived', 0)` en ambas. Evidencia: `tests/Feature/MapTest.php` (5 tests, verde) |
| `RP-MAP-003` | `PARCIAL` | `MapView` mantiene una sola `selectedCountry`, por lo que sólo renderiza un `MapCountryPopupView`; un overlay transparente sobre el fondo la pone en `nil` al tocar. La implementación coincide, pero no existe test de interacción que demuestre apertura única y cierre por fondo. |
| `RP-MAP-004` | `PARCIAL` | Los marcadores se crean únicamente con `ForEach(mapViewModel.mapItems)` y API ya excluye archivados; si el array está vacío se muestra `map.empty.title`, localizado en español e inglés. Falta un test de presentación/decodificación que cubra país con cero activos y empty state. |
| `RP-MAP-005` | `PARCIAL` | `ListView` conserva `crushListViewModel`, `searchText` y filtros como `@State` mientras empuja `MapView` en el mismo `NavigationStack`; al volver sólo recarga la lista y no resetea filtros. La ruta coincide, pero no hay test de navegación que pruebe la conservación observable. |

#### Archivar y restaurar

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-ARC-001` | `CONFORME` | **Incumplimiento real encontrado y corregido (2026-09-19):** `CrushController::archived()` no ofrecía `search`/`gender_id`/`ducky_id`/`ever_matched`/`sort_by`, sólo `limit`/`page` — la regla exige la misma capacidad de consulta que la lista de activos. Corregido extrayendo `applyListFiltersAndSorting()`, reutilizado por `all()`/`archived()`; `all()` verificado byte-idéntico en comportamiento (`ListCrushesFilterTest`/`CrudTest` sin cambios). Evidencia: `tests/Feature/Relationship/ArchivedCrushListParityTest.php` (6 tests, verde) |
| `RP-ARC-002` | `CONFORME` | `RecentActivityLoggingTest::edits_archive_delete_and_retries_do_not_log_activity` |
| `RP-ARC-003` | `CONFORME` | Por diseño: no existe fila de alta "pendiente" en API, un `PATCH` sobre un `crushId` no confirmado simplemente 404 |

#### Eliminar un Crush

Autoridad: `VaultCrushDeletionCoordinator` (iOS), coordinación con baja funcional de API (no trazada específicamente para Crush individual en esta sesión, a diferencia de la baja de cuenta cubierta en `BRI-DEL-*`).

| ID | Estado | Evidencia / corrección |
|---|---|---|
| `RP-DEL-001` | `CONFORME` | `DeleteCrushUseCase::execute()` (transaccional, `findByIdWithLock`): soft-elimina blobs de Vault por binding antes de tocar estructura, luego experiencias/períodos/crush. Evidencia: `CrushLifecycleTest::test_manual_crush_deletion_soft_deletes_related_entities` |
| `RP-DEL-002` | `CONFORME` | `DeleteCrushUseCase` elimina el grafo completo de esa ficha, sin la conservación estructural de la baja de cuenta (contratos distintos, confirmado por lectura de ambos flujos) |
| `RP-DEL-003` | `CONFORME` (API) / `PARCIAL` (iOS) | API: intención se persiste antes de mutar estructura (mismo mecanismo que `RP-DEL-001`). iOS: `VaultCrushDeletionCoordinatorTests` cubre fixtures locales pero la persistencia durable previa a la llamada de red no se confirmó símbolo a símbolo |
| `RP-DEL-004` | `CONFORME` (API) / `PARCIAL` (iOS) | **API confirmado (2026-09-19):** `CrushController::delete()` distingue 404 genuino (nunca existió) de 200 idempotente (ya soft-eliminado, vía `Crush::onlyTrashed()->exists()`, sin depender del binding). Evidencia: `VaultEntityBindingTest::repeating_a_crush_delete_after_a_lost_response_is_idempotent_and_never_needs_the_binding_again` + nuevo `CrushLifecycleTest::test_deleting_a_crush_that_never_existed_returns_a_genuine_404`. iOS: ver `RP-DEL-003` |
| `RP-DEL-005` | `CONFORME` | Mismo mecanismo que `RP-DEL-001`/`004`: confirmación funcional antes de limpieza local (ya demostrado también para baja de cuenta completa en `BRI-DEL-*` de §5) |
| `RP-DEL-006` | `CONFORME` (iOS) / `INCUMPLIDA` (Android) | iOS expone el borrado desde `CrushRowView` en la lista activa y no desde `CrushProfileView`. Android mantiene `deleteCrush()` y su confirmación dentro de `CrushProfileScreen`, contrario a la ubicación definida; debe moverse a la fila activa y desaparecer de la ficha y de archivados. |

**Resumen de estados — Account (76 IDs, 77 filas de matriz; actualizado tras aceptar `API-ACCOUNT-10`, 2026-09-19).** `CONFORME`: 70 · `PARCIAL`: 3 (`RP-CUE-005` por columnas inertes, `BR-3.03` por normalización/cifrado pendiente, `RP-PER-003` por falta de prueba de persistencia del idioma) · `INCUMPLIDA`: 3 (`RP-VLT-005`/`RP-CUE-009` por cifrado del email, propietario `user-information`; `RP-APL-014` por falta del aviso previo del email Apple, fases `IOS-ACCOUNT-01/02`) · `NO DEMOSTRADA`: 0 · `CONTRADICCIÓN DOCUMENTAL`: 0 · `NO APLICABLE`: 1. El recuento es por fila, no por ID individual.

**Resumen de estados — Crush (58 IDs canónicos; clasificación conservadora por ID y por la plataforma peor situada).** `CONFORME`: 40 · `PARCIAL`: 9 (`RP-CRU-007`, `RP-CRU-020`, `RP-AGE-015`, `RP-FOT-006`, `RP-MAP-003`, `RP-MAP-004`, `RP-MAP-005`, `RP-DEL-003`, `RP-DEL-004`) · `INCUMPLIDA`: 9 (`BR-4.01`, `RP-CRU-001`, `RP-CRU-004`, `RP-CRU-008`, `RP-CRU-009`, `RP-CRU-013`, `RP-CRU-015`, `RP-CRU-023`, `RP-DEL-006`) · `NO DEMOSTRADA`: 0 · `NO APLICABLE`: 0. Cuando una regla difiere por plataforma, el resumen usa el peor estado; la fila conserva el detalle.

Ambos recuentos son un apoyo de lectura, no un criterio de cierre: la fuente de verdad es la fila individual de cada ID en las tablas de arriba.

## 7. Compatibilidad histórica que debe auditarse y retirarse

La búsqueda debe cubrir como mínimo:

- `legacy`, `old`, `previous`, `migration`, `migrate`, `fallback`, `compat`, `v1`, `state_id`, `RelationshipState`, `secure_description` y campos privados retirados;
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

**Estado:** iniciada, con contraste ejecutado sólo para Account + Crush. Ver «Matriz BRI-00B — Account y Crush» arriba para los 76 IDs de Account y 58 IDs de Crush con estado, evidencia y propietario. Crush ya no conserva ningún ID `NO DEMOSTRADA`: la auditoría fresca del 2026-09-19 reclasificó los 19 pendientes como 13 conformes, 5 parciales y 1 incumplido. Los demás dominios (`EXPERIENCES`, `RELATIONSHIPS`, `TIMELINE`, `SYNC`, `VAULT`, `STATISTICS`, `UX`, `DATA_LIFECYCLE`, `PRIVACY`) ya están finalizados funcionalmente pero siguen sin contraste; `BRI-00B` no se declara completa hasta cubrirlos.

Para cada regla referenciada desde §6:

- localizar autoridad API, preflight iOS, consumidores y tests mediante índices frescos;
- registrar `CONFORME/PARCIAL/INCUMPLIDA/NO DEMOSTRADA` con símbolos y evidencia;
- recorrer todas las rutas alternativas que puedan eludir la regla;
- completar el inventario de compatibilidad del §7;
- no corregir nada todavía.

### `BRI-00C` — revisión funcional

Revisar con el usuario, una por una, las discrepancias de implementación demostradas. Las definiciones funcionales ya se revisan antes de `BRI-00B`; cualquier decisión nueva se actualiza primero en el documento de reglas del dominio, enlazado desde `PRODUCT.md`. Congelar los resultados antes de abrir fixes.

### Fases correctivas

Se crean sólo desde la matriz congelada y después de `BRI-00C`. En Account/Crush ya se autorizaron y ejecutaron varias fases; los candidatos del resto del catálogo continúan bloqueados hasta que sus reglas sean revisadas. Candidatos ya demostrados para el resto del catálogo:

1. `API-BRI-01`: separar anonimización estructural de destrucción Vault y corregir baja de cuenta, purga, actividad y estadísticas post-baja.
2. `API-BRI-02`: retirar compatibilidad/runtime legacy de API y su cobertura.
3. `IOS-BRI-01`: retirar compatibilidad/runtime legacy de iOS y su cobertura.
4. lotes posteriores por dominio, con una regla o grupo inseparable por fase.

Candidatos específicos del subalcance Account/Crush, detallados en los planes locales de API e iOS:

5. `API-CRU-01` **implementada; recepción con validaciones pendientes** (ver `api/docs/plans/active/crush-rule-compliance/README.md`): corrigió alta, edición y respuestas del perfil, Ducky, ratings y `age_as_of`. El siguiente lote independiente es `API-CRU-RED-01B/01B`, que retira los consumidores restantes de `gender_id`. `API-CRU-RED-02/02` (mutación atómica de `met_at`) sigue pendiente y bloqueada por el WIP de `relationship-redesign`.
6. `API-ACCOUNT-04a/05/06/07/08/09/10` **implementadas y recibidas con el estado indicado en el plan local**. `API-ACCOUNT-10` corrigió los dos bypasses del lock OTP: `RP-VER-002` y `BR-3.08` pasan a `CONFORME`, incluida la carrera MySQL/InnoDB y la respuesta anti-enumeración. `API-ACCOUNT-04b` (cifrado/lookup de email) permanece en `user-information`.
7. `IOS-CRU-01` **implementada; evidencia automatizada reportada y no repetida en la recepción**: `CrushEditViewModel.buildPayload`/`addFullParameters`, `CrushFormModel.toAPIRequest` y `EditCrushFormModel.toAPIRequest`/`ageAndZodiacParameters` ya no incluyen `gender_id` ni `age_as_of` en el payload de red. Claude reportó `xcodebuild build-for-testing`/`test-without-building` verdes sobre las suites afectadas y `git diff --check` limpio.
8. `IOS-ACCOUNT-01` **con implementación aceptada y validaciones pendientes**: los tests de persistencia del idioma y aviso previo del email Apple están escritos, pero el target no compila por un archivo de `relationship-redesign` aún fuera de `project.pbxproj`. `IOS-ACCOUNT-02` permanece bloqueada hasta confirmar el VERDE/RED esperado y aceptar la fase de tests.
9. `API-CRU-RED-03/03`, `IOS-CRU-02` y el slice correspondiente de `android-parity` deben hacer obligatoria la experiencia inicial completa. Este corte es transversal y no se inicia mientras el contrato de alta siga siendo modificado por `relationship-redesign`; primero se congelan request, snapshot durable, IDs, bindings y tests de los tres clientes.
10. Android debe retirar el borrado integral desde `CrushProfileScreen` y ofrecerlo sólo desde la fila activa conforme a `RP-DEL-006`; el cambio pertenece a `android-parity`, no al plan local de API.
11. `IOS-CRU-03` debe corregir `RP-CRU-023`: nombre y profesión a 191 caracteres, redes a 255 y WhatsApp a 20 dígitos con prefijo de catálogo y borrado conjunto, todo con tests de frontera y persistencia Vault.
12. `IOS-CRU-04` debe convertir en evidencia automatizada las cinco filas parciales descubiertas en esta pasada: composición completa API+Vault (`RP-CRU-020`), exclusión de fotos en recovery (`RP-FOT-006`) y comportamiento observable del mapa (`RP-MAP-003..005`). Es una fase de tests salvo que el RED revele una desviación real.

`BRI-PLAN-001` (§5) está resuelto: `account-rule-compliance`/`crush-rule-compliance` quedaron adoptados en `AGENTS.md` §3 y `api/AGENTS.md`, y `crush-age-birthday` se eliminó (su antecedente vive ahora en `crush-rule-compliance/README.md`). Las fases Account/Crush descritas como ejecutadas ya no están bloqueadas por `BRI-00C`; su estado real y trabajo restante se mantiene en sus planes locales.

Los fixes relacionales todavía abiertos permanecen en `relationship-redesign` hasta su aceptación. No se duplican aquí. Los correctivos de cifrado/minimización del email de cuenta (`RP-VLT-005`, `RP-CUE-009`, `RP-PER-008`) permanecen en `user-information`, no en BRI. El contrato `birth_date` completo de `BRI-ACC-003` ya está resuelto (decisión 6); la sustitución de los tests obsoletos de `birth_year` es ejecución de `account-rule-compliance`, no una decisión pendiente.

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

En la auditoría coordinadora del 2026-09-19 se reindexaron los worktrees actuales: API (`7287` nodos, generación `2026-09-19T15:24:48Z`) e iOS (`12154` nodos, generación `2026-09-19T16:04:41Z`). Los archivos API citados y los símbolos iOS de formularios, edad, búsqueda, mapa, medios y tests reportaron metadatos frescos; `CrushViewModel.swift` conserva tres rangos parciales ajenos a la carga/composición del perfil y se leyeron directamente. Product se contrastó directamente porque la documentación Markdown está excluida del grafo. El resto del catálogo (fuera de Account/Crush) sigue sin esta comprobación.

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
5. ~~`BRI-PLAN-001`~~ — **Resuelto (2026-09-19).** `account-rule-compliance` y `crush-rule-compliance` adoptados en `AGENTS.md` §3 y `api/AGENTS.md`; `crush-age-birthday` eliminado, con su contrato documentado como antecedente en `crush-rule-compliance/README.md`.
6. ~~`BRI-ACC-003`~~ — **Resuelto (2026-09-19).** El perfil de cuenta conserva `birth_date` completo; no se migra a `birth_year`-only. Los tests obsoletos se sustituyeron durante `API-ACCOUNT-08`.
7. ~~`RP-FOT-007`~~ — **Resuelto (2026-09-19).** Al llegar a cinco fotos se bloquea la selección al hueco restante; no se reemplaza automáticamente ninguna foto existente. La implementación y sus tests están descritos en la fila de la matriz.
