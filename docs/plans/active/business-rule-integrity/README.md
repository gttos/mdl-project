# Integridad de reglas de negocio

> **Estado:** primero se define y revisa el catálogo funcional por dominios, accesible desde [PRODUCT.md](../../../PRODUCT.md). El contraste de implementación queda diferido; esta consolidación no inicia `BRI-00B`, no acepta fases de código ni autoriza fixes. La futura auditoría usará los IDs del catálogo y evidencia fresca.

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
- Apple conserva el email acreditado por el proveedor sin OTP MDL. Los [dos recorridos para añadir un método de acceso](../../../rules/ACCOUNT.md#metodos-acceso-apple) están definidos y pendientes de implementación completa; sólo el tratamiento de dos cuentas MDL distintas ya creadas permanece abierto en DEC-01.
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

La futura matriz vive en este plan y referencia cada ID `RP-*`/`BR-*`; no copia las reglas. Todavía no se ha iniciado esa matriz. Los hallazgos previos de §5 se conservan como entradas de investigación, sin atribuirles validación nueva.

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

Sólo después de que el usuario dé por finalizados los documentos de reglas y acuerde iniciar el contraste, para cada regla referenciada desde §6:

- localizar autoridad API, preflight iOS, consumidores y tests mediante índices frescos;
- registrar `CONFORME/PARCIAL/INCUMPLIDA/NO DEMOSTRADA` con símbolos y evidencia;
- recorrer todas las rutas alternativas que puedan eludir la regla;
- completar el inventario de compatibilidad del §7;
- no corregir nada todavía.

### `BRI-00C` — revisión funcional

Revisar con el usuario, una por una, las discrepancias de implementación demostradas. Las definiciones funcionales ya se revisan antes de `BRI-00B`; cualquier decisión nueva se actualiza primero en el documento de reglas del dominio, enlazado desde `PRODUCT.md`. Congelar los resultados antes de abrir fixes.

### Fases correctivas

Se crean sólo desde la matriz congelada. Primeros candidatos ya demostrados:

1. `API-BRI-01`: separar anonimización estructural de destrucción Vault y corregir baja de cuenta, purga, actividad y estadísticas post-baja.
2. `API-BRI-02`: retirar compatibilidad/runtime legacy de API y su cobertura.
3. `IOS-BRI-01`: retirar compatibilidad/runtime legacy de iOS y su cobertura.
4. lotes posteriores por dominio, con una regla o grupo inseparable por fase.

Los fixes relacionales todavía abiertos permanecen en `relationship-redesign` hasta su aceptación. No se duplican aquí.

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

Al iniciar este plan, el índice API seguía en `b20df4b…` y el de iOS en `93e925a…`; el reindexado API fue rechazado dos veces porque `codebase-memory-mcp` detectó otra generación activa no verificada. Por eso la matriz del §6 no afirma una auditoría exhaustiva del código actual.

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
