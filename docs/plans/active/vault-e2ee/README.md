# Private Journal Vault (E2EE)

> **Autoridad funcional:** [Vault](../../../rules/VAULT.md), [Baja y ciclo de vida](../../../rules/DATA_LIFECYCLE.md) y [Tareas automáticas y plazos](../../../rules/DATA_LIFECYCLE.md#tareas). Las referencias históricas de ejecución de este plan no establecen requisitos de compatibilidad para el lanzamiento. Baja de cuenta conserva estructura seudonimizada y destruye contenido privado según ese catálogo; no se equipara al borrado manual del Crush.

> **Estado:** activo. El baseline de cuentas nuevas, el borrado dirigido de experiencia/relación, `API-14`, `iOS-09.8`, `API-14R`, `iOS-09.8R`, `iOS-09.8S`, `iOS-09.8U`, `iOS-09.8V`, `API-14S` e `iOS-09.8W` están validados. La migración física del único usuario legacy terminó con el Vault validado/activado y el historial se recuperó correctamente en entornos limpios por contraseña y por frase de 24 palabras. `CROSS-02` y los backups coordinados de base de datos y objetos Vault están validados. `API-05.3b` está desplegada y `iOS-09.8T-R` + `CROSS-03` están validadas físicamente con un único dispositivo activo: las respuestas funcionales ya no traen contenido privado y iOS lo resuelve sólo desde Vault. Los cortes `API-15A`, `API-16` y `API-16b` retiraron las columnas privadas legacy de producción; cualquier cleanup restante debe tener autorización destructiva separada.
> **Proyectos:** Product, API e iOS.

## Objetivo

El contenido privado de crushes, experiencias y relaciones se cifra en iOS y puede recuperarse en otro dispositivo sin que API/S3 reciban la VMK o el plaintext. Ningún dato legacy se elimina hasta demostrar que el historial completo del usuario antiguo existe en Vault, se recupera desde un dispositivo limpio y la app funciona sin depender de columnas privadas de API.

## Alcance

- Baseline nuevo con bindings opacos, blobs versionados y recovery.
- Reconstrucción completa del único usuario legacy desde los datos funcionales todavía conservados.
- Retirada de inputs/outputs privados de API después de validar recovery legacy.
- Vaciado y posterior eliminación de columnas legacy mediante gates separados y autorización explícita.
- Borrado coordinado de crush, fotos locales y baja/reset integral antes del cierre final.

Fuera de alcance:

- backup remoto de fotos/videos;
- migrar otros usuarios inexistentes o inventar compatibilidad genérica no requerida;
- exponer una migración recurrente o una superficie de producto para cuentas nuevas;
- interpretar ciphertext en API;
- ejecutar cleanup, `DROP`, reset, deploy o migración real desde una sesión de documentación/implementación sin autorización operativa específica.

## Estado verificado

| Área | Estado | Evidencia/límite |
|---|---|---|
| Cuenta/Vault nuevos | Validado | Alta, edición y confirmación remota desde el dispositivo activo; recovery correcto al sustituirlo. |
| Binding opaco y blob con token | Validado | Token obligatorio en altas nuevas; ownership, idempotencia y rollback cubiertos. |
| Borrado experiencia/relación | Validado | Un único `DELETE` funcional, sin escaneo global; recovery no revive lo eliminado. |
| Binding de entidades legacy (`API-14`) | Validado | `POST /vault/migration-bindings` compara el manifest exacto contra el inventario activo y crea/reutiliza bindings atómicamente, sin plaintext ni tokens en respuestas/logs. |
| Migración legacy iOS (`iOS-09.8` + `iOS-09.8R`) | Validada físicamente | Staging tokenizado consumió API-14, la fuente canónica API-14R reemplazó Timeline, el pipeline alcanzó validación/activación final y el historial se reconstruyó en entornos limpios. |
| Fuente canónica de migración legacy (`API-14R` + `iOS-09.8R`) | Implementada y validada con tests | `GET /vault/migration-source` y el manifest comparten el mismo predicado migrable; iOS transforma el grafo plano directamente en staging/manifest y muestra fallos de forma segura. |
| Reconciliación de staging legado (`iOS-09.8S`) | Validada dentro de la migración física | Reutilizó/reconcilió el staging sin impedir que el flujo real llegara a sync y activación. |
| Corrección de fuente/progreso (`iOS-09.8U`) | Validada dentro de la migración física | Acepta el timestamp con seis microsegundos que Eloquent emite en el contrato canónico, conserva el fail-closed y reserva 100 % para la finalización real, incluida activación. |
| Diagnóstico de decodificación de fuente (`iOS-09.8V`) | Implementada; fecha efectiva resuelta por `API-14S` | El schema real permite `updated_at` nulo; iOS lo transporta sin relajar otros campos y la fuente temporal materializa la revisión con la prioridad canónica antes de construir el snapshot. |
| Fecha efectiva de registros legacy sin actualización | Validada dentro de la migración física | Sólo en la fuente temporal: `updated_at`, luego `created_at`, luego una única hora actual UTC de la respuesta; no cambia filas legacy ni usa la hora actual por entidad. |
| Respuesta de manifest y diagnóstico por etapa (`iOS-09.8W`) | Validada físicamente | El decoder consume camelCase real, los fallos se clasifican por etapa y ninguna representación de UI/logging deriva de `LegacyMigrationFailure.identifiers`; el flujo real llegó a activación y recovery. |
| Recovery del historial legacy | Validado | Contraseña y frase de 24 palabras reconstruyen el historial migrado en entornos limpios. |
| Outputs/inputs privados de API | Validados físicamente | `API-05.3b` retiró los campos privados del contrato; `API-15A`, `API-16` y `API-16b` retiraron también sus columnas legacy en producción. Los bindings de API-13 permanecen. |
| Compatibilidad iOS con el corte privado (`CROSS-03`) | Validada físicamente | Listado, Home, perfil y Timeline toleran su ausencia en HTTP y resuelven nombre/lugar/descripción/nota sólo desde el Vault local válido. |
| Actividad reciente de Home | Implementada; validación física pendiente | API registra las altas confirmadas de Crush, Experience y una Relationship posterior, conserva sólo referencias estructurales y no duplica la relación inicial. Editar una Relationship existente —aunque cambie `state_id`— no genera actividad, conforme a [Actividad de Home](../../../rules/TIMELINE.md#home). La cobertura automatizada de `API-PENDING-05` está completa; queda comprobar el feed físicamente tras despliegue. |
| Columnas privadas legacy de API | Validadas físicamente | `secure_description` y las 13 columnas restantes de contenido privado ya no existen en producción; sus datos residen exclusivamente en Vault. La recuperación de datos previos sólo es posible desde los snapshots. |
| Superficie Debug de migración legacy | Implementada; pendiente de integración remota | `iOS-09.8T-R` retira el botón Debug y los componentes exclusivos de la operación única. |
| Alta normal independiente de migración legacy | Implementada; pendiente de validación física | `VaultAccountSetupCoordinator` sustituyó el pipeline legacy: inicializa/activa/sincroniza por contratos normales y no llama a rutas de migración única. |
| Sincronización entre dos dispositivos activos | Diferida fuera de beta | La prueba física confirmó que el origen no descarga una revisión subida desde el dispositivo recuperado durante uso normal. La beta adopta un único dispositivo activo por Vault; no se implementa reconciliación incremental ni conflictos multi-device ahora. |
| Cleanup legacy | Pendiente | `CleanupLegacyDataJob` es un placeholder y `migrate-complete` no autoriza limpieza. |
| Borrado de crush/fotos (`API-17` + `iOS-09.9`) | Implementado; validación coordinada pendiente | API resuelve el grafo y blobs bajo transacción; iOS limpia entries, notas y `LocalMedia/{crushId}` sólo tras confirmación durable. Faltan despliegue/recorrido físico coordinado y una prueba de concurrencia representativa donde corresponda. |
| Baja/reset integral (`API-11` + `iOS-12`) | Implementado; validación física pendiente | API confirma la baja y expone la confirmación durable ante respuesta perdida; iOS elimina Vault/Keychain/fotos locales sólo después de confirmación. Falta repetir el recorrido físico completo, incluida una cuenta con Vault activo y el relanzamiento tras una respuesta ambigua. |
| Exclusión de fotos locales de backups (`iOS-13`) | Implementada; validación manual pendiente | `LocalMedia/`, sus subcarpetas por Crush y cada foto se marcan para excluirse del backup del sistema. Antes de lanzamiento hay que comprobar un backup real de dispositivo. |
| Límites y cuota remota (`RP-VLT-027..031`) | Contrato definido; implementación pendiente | API e iOS deben aplicar 65.536 bytes por objeto, 25 MiB lógicos por cuenta, aviso al 80 %, reserva idempotente, observabilidad sin plaintext y caducidad a las 24 horas. La API debe liberar la cuota al vencer y limpiar objetos y metadata incompletos; iOS conserva la intención durable y solicita una capacidad nueva al reintentar. |

## Reglas compartidas y estado

### GWT-VLT-36 — bindings para entidades existentes

**Estado:** validado. El flujo físico aceptó el manifest, sincronizó el conjunto, terminó con el Vault validado/activado y se recuperó correctamente en entornos limpios por contraseña y por frase de 24 palabras.

**Dado** un Vault del propietario y un inventario exacto de crushes, experiencias y relaciones activas ya existentes,

**cuando** iOS presenta el tipo, ID funcional y token opaco de cada entidad para migración,

**entonces** API valida ownership/existencia, crea o reutiliza bindings idempotentes y rechaza cualquier omisión, duplicado, cruce de usuario o token contradictorio sin mutación parcial.

El request no contiene plaintext, VMK ni ciphertext. La API no inventa tokens ni permite vincular una entidad eliminada/ajena.

### GWT-VLT-37 — reconstrucción legacy completa y no destructiva

**Estado:** validado. La reconstrucción física del usuario legacy terminó y el historial migrado se reconstruyó correctamente desde remoto en entornos limpios.

**Dado** el único usuario legacy con sus columnas aún intactas y un backup operativo confirmado,

**cuando** iOS obtiene de API el inventario canónico migrable por ID funcional,

**entonces** crea un snapshot v2 completo y tokenizado por cada crush, experiencia y relación, sube/verifica todas las revisiones nuevas y conserva el origen y las revisiones remotas anteriores hasta demostrar el conjunto nuevo.

La fuente canónica contiene cada entidad no eliminada cuyo crush no está anonimizado, incluida la historia archivada, exactamente una vez y con los datos legacy necesarios para su snapshot. La cantidad e IDs de cada tipo deben coincidir exactamente con el inventario que API valida para bindings. Timeline es una proyección de interfaz y no es una fuente válida de migración. Para la revisión del snapshot, la fuente temporal materializa `updated_at`, después `created_at` y, si ambos faltan, una única hora UTC por respuesta; varias experiencias/relaciones del mismo crush no colisionan.

La migración es una operación manual, única y temporal del único usuario legacy. El botón Debug no forma parte del producto ni se activa para cuentas nuevas. Tras `CROSS-02` validada, se retiran de iOS el botón, el orquestador y el staging de migración, y de API las rutas exclusivas de migración (`GET /vault/migration-source`, `GET /vault/migration-status`, `POST /vault/migration-bindings` y el `migrate-complete` legacy); las altas normales continúan usando los bindings de API-13.

### GWT-VLT-38 — sustitución segura del conjunto remoto

**Estado:** validado. El flujo físico alcanzó la activación final, que exige inventario remoto exacto, y el conjunto activado se recuperó correctamente. La prueba deliberada de interrupción/reanudación queda como comprobación complementaria, no como bloqueo de `CROSS-02`.

**Dado** que todas las revisiones nuevas se descargaron, descifraron y compararon contra staging,

**cuando** se retiran las revisiones antiguas/sobrantes,

**entonces** la operación usa una lista durable e idempotente, no elimina blobs corruptos/no atribuibles sin aprobación humana, conserva la ventana de recuperación/backup y finaliza con un inventario remoto exacto.

Un crash entre subida, retiro y activación reanuda la misma migración; nunca publica un conjunto parcial.

### GWT-VLT-39 — recovery físico del usuario legacy

**Estado:** validado por contraseña y por frase de 24 palabras. La prueba posterior de edición confirmó que una copia recuperada puede subir su propia revisión, pero el origen no la descarga durante uso normal. La beta queda definida como un único dispositivo activo por Vault; por tanto, no se exige propagación entre ambos como criterio de este gate.

**Dado** el conjunto migrado y activado,

**cuando** la cuenta inicia sesión en un dispositivo/namespace limpio y recupera el Vault,

**entonces** Home, perfiles y Timeline reconstruyen todos los IDs y valores privados correctos, sin extras, faltantes, reasignaciones ni borrados remotos.

La validación se completó con contraseña y con frase de 24 palabras, comprobando el mismo historial migrado.

### GWT-VLT-40 — corte del contrato privado de API

**Dado** que GWT-VLT-39 está validado,

**cuando** se despliega el corte contractual,

**entonces** API deja de aceptar y devolver nombre/contacto/redes/cumpleaños y lugar/descripción en rutas funcionales, e iOS sigue mostrando ese contenido únicamente desde Vault.

Las columnas permanecen todavía como red de seguridad; no se vacían en esta fase.

### GWT-VLT-41 — cleanup por gates separados

**Dado** backup, recovery legacy y corte contractual validados,

**cuando** el usuario autoriza el cleanup,

**entonces** primero se ejecuta un dry-run/inventario, después se ponen a `NULL` sólo los campos privados legacy previstos y se repite recovery/uso normal. El `DROP` de columnas ocurre en otra autorización irreversible posterior, nunca en el mismo despliegue.

`migrate-complete` no dispara cleanup automáticamente y una métrica/conteo distinto del esperado bloquea el proceso.

### GWT-VLT-42 — borrado integral restante

**Dado** un crush o una cuenta con Vault y fotos locales,

**cuando** se confirma su eliminación,

**entonces** API/iOS coordinan estructura, bindings, blobs, entries/notas, encapsulaciones/devices cuando corresponda y `LocalMedia`, con progreso durable, retry y sin falso éxito.

Si falta cualquier binding requerido del Crush o de su grafo de experiencias y relaciones, API rechaza el borrado integral completo sin mutación parcial; el retry sólo procede después de reparar la inconsistencia.

El borrado integral de Crush sigue RP-DEL-001..005. La corrección de un hito y el borrado de un período siguen RP-HIT-001..013; ningún borrado de período reabre automáticamente uno anterior.

Los bindings del grafo se conservan sólo durante el soft delete y la ventana de retry/purga; se eliminan cuando la purga física de todas sus revisiones termina, no durante la baja de cuenta.

Logout, reset local y baja de cuenta son acciones distintas. No existe una operación de reset de Vault independiente de la baja: `DELETE /vault` (API) y "Reset Vault (Debug)" (iOS) quedan retirados.

**Contrato confirmado para `API-11`/`iOS-12`; implementado y validado en API, `iOS-12` pendiente:**

1. `DELETE /users` realiza de forma síncrona y como una única operación atómica y fail-closed toda la baja lógica y criptográfica de la cuenta: sólo responde éxito cuando el estado remoto queda coherente; ante cualquier fallo, rollback total sin baja parcial.
2. Los blobs Vault del usuario entran de inmediato en soft-delete. Se conservan cifrados durante la misma ventana de siete días que el borrado dirigido y luego la purga programada los elimina físicamente de S3.
3. Los bindings se conservan únicamente para localizar y completar esa purga; se eliminan al desaparecer la última revisión de cada binding, no en el momento de la baja.
4. Tokens de sesión, dispositivos Vault, recovery keys (contraseña y frase de 24 palabras) y sus encapsulaciones quedan invalidados/eliminados de inmediato; ninguno sobrevive a una baja exitosa.
5. iOS sólo después de recibir el 200 de `DELETE /users`, o la confirmación `completed` del mecanismo del punto 8 si esa respuesta se perdió, elimina SQLite Vault, VMK y material Keychain, fotos/`LocalMedia`, notas, operaciones pendientes y preferencias sensibles. Si ninguna de las dos vías confirma la baja, iOS no borra nada local.
6. La baja es irreversible: no existe restauración parcial. No se admite reactivar sólo `users` ni estados mixtos de crushes anonimizados con experiencias/relaciones borradas.
7. `vaults.user_id` conserva su restricción UNIQUE: existe un único Vault durante toda la vida del usuario. `POST /vault/init` debe detectar también Vaults soft-deleted y responder con un error de dominio controlado; nunca recrearlos ni producir una excepción SQL.
8. Confirmación segura ante respuesta perdida: antes de `DELETE /users`, iOS genera un secreto de un solo uso (≥256 bits), lo persiste localmente y lo envía en un header dedicado de esa petición. API persiste sólo su hash junto al receipt interno, dentro de la misma transacción de la baja -- nunca el secreto en claro ni en logs. Un endpoint separado, sin sesión/token, rate-limited y protegido exclusivamente por ese secreto, confirma únicamente si la baja quedó `completed`; un secreto incorrecto, ausente o inexistente produce siempre el mismo resultado genérico, sin exponer email, receipt ID, Vault ID ni ningún otro dato. El hash no vence por sí mismo: se conserva mientras exista el receipt interno, bajo su misma retención. No sustituye ni habilita un endpoint de estado por usuario, y ninguna ruta existente relaja su autenticación.
9. La baja de cuenta también cubre un Crush que ya estuviera soft-deleted desde antes del borrado integral (`API-17`) y que por eso nunca completó la limpieza de sus blobs Vault activos: esa limpieza corre antes del cierre final del Vault, reutilizando el mismo mecanismo ya validado para el borrado dirigido de un Crush individual. Repetir la baja sobre un grafo ya sin nada pendiente es idempotente.

Este contrato resuelve la decisión que quedaba abierta en este plan; queda fuera de `CROSS-02` y no se resuelve mediante reparaciones manuales de datos. Detalle técnico en [`product/docs/VAULT.md`](../../../VAULT.md) §6.

## Orden obligatorio

| Orden | Fase | Proyecto | Estado/entrada |
|---|---|---|---|
| 1 | `API-14` — manifest/binding de entidades legacy | API | **Validada**; contrato en `POST /api/vault/migration-bindings`. |
| 2 | `iOS-09.8` — staging legacy con token y retiro seguro de revisiones anteriores | iOS | **Validada físicamente** dentro de la migración real del usuario legacy. |
| 3 | `API-14R` — fuente canónica legacy | API | **Validada físicamente**; la migración real consumió `GET /vault/migration-source` sin depender de Timeline. |
| 4 | `iOS-09.8R` — staging desde fuente canónica | iOS | **Validada físicamente**; convirtió el inventario real y conservó los gates de manifest, sync, retiro y activación. |
| 5 | `iOS-09.8S` — reconciliación local de staging legado | iOS | **Validada físicamente**; el staging existente avanzó hasta sync/activación. |
| 6 | `iOS-09.8U` — corrección de fuente/progreso real | iOS | **Validada físicamente**; el progreso sólo llegó a finalización con activación exitosa. |
| 7 | `iOS-09.8V` — diagnóstico de decodificación real | iOS | **Validada físicamente** junto con la fecha efectiva de `API-14S`. |
| 8 | `API-14S` — fecha efectiva temporal de legacy | API | **Validada físicamente**; `updated_at → created_at → una hora UTC por respuesta`, sin mutar base ni afectar rutas normales. |
| 9 | `iOS-09.8W` — contrato real del manifest y diagnóstico seguro por etapa | iOS | **Validada físicamente**; decoder camelCase, diagnóstico seguro y activación real confirmados. |
| 10 | `CROSS-02` — backup, migración real y recovery en dispositivo limpio | Coordinada | **Validada**; migración/activación y recovery por contraseña y frase completados. Edición/reversión cross-device queda como prueba complementaria previa al despliegue del corte. |
| 11 | `API-05.3b` — corte de campos privados y rutas temporales | API | **Desplegada y validada físicamente**. Retira inputs/outputs privados y las rutas únicas, sin tocar columnas físicas ni API-13. |
| 12 | `iOS-09.8T` — retirada de superficie Debug legacy | iOS | **Incorporada en `iOS-09.8T-R`**. El botón Debug y componentes exclusivos de la operación única ya no quedan en producción. |
| 13 | `iOS-09.8T-R` — desacoplar alta normal de migración única | iOS | **Implementada y validada en rama**. El onboarding nuevo conserva init, sync y recovery sin llamar a rutas legacy. |
| 14 | `CROSS-03` — validar el corte contractual | Coordinada | **Validada físicamente**. API desplegada e iOS validada con un único dispositivo activo; el corte físico posterior cuenta con autorización destructiva expresa y snapshots previos confirmados. |
| 15 | `API-15A` — retirar `secure_description` obsoleto | API | **Desplegada y validada físicamente.** La columna no existe en producción; no migra ni transforma contenido. |
| 16 | `API-16` — eliminar las restantes columnas/código legacy | API | **Desplegada y validada físicamente.** Corte irreversible directo tras backup, sin etapa `NULL` ni canario. |
| 17 | `API-16b` — eliminar `crushes.whatsapp` restante | API | **Desplegada y validada físicamente.** La única columna de contacto que quedaba fuera de API-16 también fue retirada de forma irreversible. |
| 18 | `API-17` + `iOS-09.9` — borrado de crush/fotos | API+iOS | **Implementados con pruebas automatizadas**; quedan despliegue y validación física coordinada, incluidas fotos locales y comportamiento ante respuesta perdida. |
| 19 | `API-11` + `iOS-12` — baja/reset integral | API+iOS | **Implementados con pruebas automatizadas**; la confirmación durable ante respuesta perdida está en el contrato. Queda el recorrido físico end-to-end con Vault activo. |
| 20 | `CROSS-04` — cierre documental | Coordinada | Integrar estado canónico, retirar estos planes y verificar Legal/blueprint. |

`API-05.3b` no podía adelantarse mientras sus outputs fueran la fuente necesaria para construir los snapshots del único usuario legacy. Ese gate quedó satisfecho por `CROSS-02` y los snapshots coordinados están confirmados. La revisión de `iOS-09.8T` halló que `VaultOnboardingViewModel.runMigration()` reutilizaba `LegacyMigrationOrchestrator`, incluido `GET /vault/migration-source` y `POST /vault/migration-bindings`. `iOS-09.8T-R` sustituyó ese recorrido por el bootstrap normal y retiró las dependencias legacy; `CROSS-03` demostró físicamente que, con el corte API desplegado, iOS construye las superficies afectadas desde estructura funcional y contenido privado local de Vault. Después de los snapshots, `API-15A`, `API-16` y `API-16b` se desplegaron y validaron: las columnas privadas ya no existen y la app sigue funcionando desde Vault.

## Criterios de aceptación final

- [x] Cuentas nuevas crean/sincronizan/recoveran snapshots por entidad.
- [x] Experiencia/relación se borran con una petición funcional y retry seguro.
- [x] API registra de forma atómica e idempotente el manifest exacto de bindings para entidades existentes.
- [x] La migración legacy usa una fuente canónica, funciona con bindings/tokens de API-13 y no depende de Timeline.
- [ ] La operación única del usuario legacy ya completó staging, sync y recovery; su botón Debug está retirado y el alta normal queda desacoplada de sus rutas antes de eliminar el código compartido restante.
- [ ] El inventario funcional, staging, remoto y recuperado coincide por tipo e ID.
- [x] Contraseña y frase recuperan el historial legacy en un dispositivo limpio.
- [x] API no acepta ni devuelve campos privados funcionales.
- [x] La app sigue correcta después de retirar físicamente las columnas privadas legacy.
- [x] El `DROP` legacy directo está desplegado y validado; el backup previo es la única recuperación de datos.
- [ ] Borrado de crush, fotos, cuenta y reset cumplen GWT-VLT-42.
- [ ] Legal y blueprint reflejan el estado final, no sólo el baseline nuevo.

## Planes locales

- [`api/docs/plans/active/vault-e2ee/README.md`](../../../../../api/docs/plans/active/vault-e2ee/README.md)
- [`ios/docs/plans/active/vault-e2ee/README.md`](../../../../../ios/docs/plans/active/vault-e2ee/README.md)
