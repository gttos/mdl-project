# Paridad Android

> **Estado:** activa para una distribución Android futura y fuera de la beta activa iOS/API; no bloquea su despliegue ni su cierre. `AND-01` a `AND-04` aceptadas, y `AND-05`/alcance no bloqueado de `AND-06`/`AND-07`/`AND-08` con implementación aceptada; `AND-09` está en progreso, con correcciones accesibles/visuales aceptadas como implementación y validaciones manuales pendientes. El slice Add Crush preserva snapshots previos, corrige revisiones Vault y cubre creación durable completa. El scroll de Experience Add/Edit, la separación visual del título biométrico en Settings y la altura adaptable de las tarjetas de Home están aceptados como correcciones de layout a escala extrema; la ubicación de «Eliminar Crush» conforme a `RP-DEL-006` está aceptada con 5/5 pruebas instrumentadas y 463/463 JVM. Android ya tiene una base de Archivados que se conserva sin cambios; la aceptación de la pantalla completa y de `RP-LIS-001..004` queda para `FSR-07`, junto con las acciones legacy del Timeline, el borrado de Experience no conforme y las validaciones físicas/humanas/externas
> **Proyectos:** Product / Android; API e iOS como contratos y evidencia, no como ámbitos de implementación por defecto
> **Última actualización:** 2026-09-23

Conforme a `RP-UX-006`, ninguna mención de este plan a TalkBack, lectores de pantalla, `font scale`, escalas extremas o texto ampliado constituye trabajo pendiente, gate de aceptación ni fase futura. Las implementaciones y pruebas ya realizadas se conservan como evidencia; no se abren correctivos nuevos por esas capacidades dentro del alcance actual.

## Objetivo

Preparar Android como cliente nativo de MDL para una distribución posterior a la beta activa iOS/API, con el mismo comportamiento observable, contrato HTTP, propiedad de datos, garantías de Vault, semántica durable y calidad de referencia de iOS, usando una implementación idiomática de Android.

Paridad no significa replicar clases o gestos exclusivos de Apple. Significa que, ante el mismo estado de cuenta y las mismas acciones, ambos clientes respetan las mismas reglas y producen resultados equivalentes, salvo una excepción de plataforma documentada y aprobada aquí.

## Alcance

Incluye:

- autenticación, sesión provisional, verificación de email, recuperación de acceso y onboarding;
- Vault E2EE interoperable, recovery y separación por cuenta/dispositivo;
- escrituras durables, proyecciones pendientes, retry y reconstrucción tras relanzar;
- Crushes, Experiences, Relationships, Home, Timeline, Map, Statistics y Settings vigentes en beta;
- fotos privadas locales, App Lock, baja integral y borrados coordinados;
- paridad visual, formularios, accesibilidad, localización y evidencia automatizada/física;
- adopción de las iniciativas activas de Product que afecten a cualquier cliente nativo.

Fuera de alcance:

- nuevas funciones que no estén aprobadas para iOS/beta;
- sincronización continua entre dos dispositivos Vault activos;
- backup remoto de fotos;
- activación del servicio LLM o envío de plaintext del Vault;
- reescribir API o iOS salvo que una incompatibilidad contractual demostrada requiera un plan y autorización propios;
- modificar Legal sin alcance y revisión legal explícitos.

## Contrato de paridad

La aceptación se evalúa en seis dimensiones inseparables:

| Dimensión | Resultado exigido |
|---|---|
| Comportamiento | Mismas reglas, transiciones, validaciones y resultados de producto. |
| HTTP | Mismos endpoints vigentes, semántica de errores, idempotencia y ausencia de campos privados. |
| Privacidad | Mismo contenido E2EE, mismas fronteras de plaintext y fotos sólo locales en beta. |
| Durabilidad | Misma confirmación local, proyección, retry, relanzamiento y aislamiento entre cuentas. |
| Experiencia | Jerarquía visual y estados equivalentes con navegación, accesibilidad y patrones propios de Android. |
| Evidencia | Tests y recorridos físicos proporcionales al riesgo, incluida interoperabilidad iOS/Android. |

## Escenarios compartidos de aceptación

### GWT-AND-01 — comportamiento equivalente

**Dado** el mismo estado estructural y privado válido de una cuenta,

**cuando** la persona ejecuta en Android una acción disponible en la beta iOS,

**entonces** obtiene el mismo resultado funcional, las mismas restricciones y el mismo estado final, salvo una desviación de plataforma registrada en este plan.

### GWT-AND-02 — frontera privada

**Dado** un alta, edición, lectura o borrado con nombre/contacto/redes/cumpleaños o lugar/descripción,

**cuando** Android procesa la acción,

**entonces** el plaintext permanece en el dispositivo, HTTP transporta sólo estructura, tokens opacos y ciphertext autorizados, y ningún log, backup o mensaje técnico expone contenido íntimo.

### GWT-AND-03 — escritura durable

**Dado** un formulario válido con conectividad disponible, ausente o ambigua,

**cuando** la persona guarda,

**entonces** Android persiste primero la intención cifrada, proyecta el resultado local, clasifica el estado visible y reanuda la misma operación idempotente tras relanzar sin duplicar ni fingir éxito remoto.

### GWT-AND-04 — recovery interoperable

**Dado** un Vault válido creado o actualizado por iOS,

**cuando** la cuenta se recupera en una instalación Android limpia mediante contraseña o frase de 24 palabras,

**entonces** Android reconstruye exactamente el inventario activo y sus valores privados; la recuperación inversa de un Vault escrito por Android ofrece el mismo resultado en iOS.

La beta conserva un único dispositivo activo: esta prueba representa sustitución del dispositivo, no edición simultánea.

### GWT-AND-05 — ciclo de vida Android

**Dado** un flujo en curso o un estado privado visible,

**cuando** ocurre rotación, cambio de tamaño, background, process death, pérdida de red o navegación atrás,

**entonces** no se filtra contenido, no se duplica una acción y el estado recuperado coincide con la frontera durable confirmada.

### GWT-AND-06 — borrado y backup

**Dado** un crush o una cuenta con Vault, operaciones pendientes y fotos locales,

**cuando** se confirma su eliminación o el sistema ejecuta backup/transferencia de dispositivo,

**entonces** se respeta el contrato canónico de confirmación y limpieza, y tokens, claves, ciphertext local y fotos quedan fuera de cualquier backup no autorizado.

### GWT-AND-07 — evidencia de release

**Dado** que todas las superficies declaradas alcanzaron paridad,

**cuando** se evalúa un candidato de beta Android,

**entonces** existen pruebas unitarias, de persistencia/cripto, de contrato, de UI y recorridos físicos que cubren cada regla crítica, sin sustituir evidencia Android por el mero hecho de que iOS ya esté validado.

## Interacción entre proyectos

| Proyecto | Responsabilidad |
|---|---|
| Product | Define reglas, GWT, alcance de beta, excepciones de plataforma y orden global. |
| Android | Implementa la paridad con componentes nativos y conserva su evidencia y snapshot técnico local. |
| API | Mantiene el contrato estructural, idempotente y sin plaintext ya vigente; sólo cambia mediante fase propia si aparece una incompatibilidad real. |
| iOS | Aporta evidencia del comportamiento actual y vectores interoperables; no es fuente funcional por encima de Product. |
| LLM | Permanece desconectado y fuera de alcance. |

## Cadencia de avance iOS → Android

Mientras `android-parity` permanezca activo, es esperado que iOS vaya por delante. La diferencia se reduce de forma controlada:

1. Product o el plan maestro de la iniciativa afectada fija primero la regla compartida.
2. iOS puede implementar y validar la fase antes que Android. Sólo una fase aceptada y reconciliada se usa como evidencia de referencia.
3. El coordinador registra en este plan y en el plan local Android qué superficie/regla nueva debe alcanzar paridad, sin copiar GWT ni abrir una spec paralela.
4. Android parte de su arquitectura y working tree vigentes, traduce el resultado a componentes nativos y aporta tests propios.
5. La fase Android se acepta por separado. Que iOS esté validado reduce incertidumbre, pero no demuestra Android.

No se copia código iOS de manera literal ni se sigue comportamiento todavía no aceptado. Cuando el contrato ya está congelado, iOS y Android pueden avanzar en paralelo sólo con autorización y propiedad de archivos/proyectos claramente separada. Después de cerrar `android-parity`, toda nueva iniciativa compartida debe incluir Android como proyecto afectado desde el inicio.

## Orden global

| Orden | Resultado | Estado |
|---|---|---|
| 0 | Gobernanza, documentos canónicos y planes de paridad | Completado el 2026-09-11 |
| 1 | Baseline Android seguro, aislado de producción y verificable | Completado el 2026-09-11 |
| 2 | Sesión, verificación de email y bootstrap de cuenta/Vault | Aceptado el 2026-09-12; el harness Hilt y su consumidor Compose productivo con `HiltAndroidRule` quedaron validados en `AND-09`; queda cobertura física contra API/mailer real |
| 3 | Vault Android e interoperabilidad criptográfica | Aceptado el 2026-09-12; contrato criptográfico y HTTP congelado con evidencia automatizada; validaciones físicas diferidas al gate `AND-09` antes de beta externa o lanzamiento |
| 4 | Motor de escritura durable y proyecciones | Aceptado el 2026-09-12; motor completo y slice real de Ratings con estado/acciones visibles; la regresión instrumentada y el grafo Compose/Hilt productivo pasaron en `AND-09`; queda el recorrido físico offline→relanzar→reconectar |
| 5 | Corte completo al contrato privado y fotos locales | Implementación aceptada el 2026-09-13; la incompatibilidad OkHttp/`mockwebserver` y el harness Hilt quedaron corregidos y la instrumentación pasó en `AND-09`; queda biometría física |
| 6 | Flujos de dominio y Timeline | Implementación aceptada el 2026-09-13 para el alcance no bloqueado de edición/borrado de Crush y Experience/Timeline; migración Room v2→v3 y cola/atomicidad Vault ejecutadas en instrumentación dentro de `AND-09`. El slice Add Crush está aceptado con validaciones pendientes: selector nativo, bundle Vault durable, proyección/descarte, corrección con revisión nueva y lectura compatible de snapshots anteriores; evidencia JVM reportada 450/450 | Pendientes funcionales: las acciones del Timeline del perfil usan condiciones legacy y `ExperienceEditScreen` ofrece el borrado incondicionalmente, sin distinguir `RP-EXP-019..021`; se corrigen con el contrato relacional final, no se aceptan por los tests de layout. Persisten validaciones físicas/externas |
| 7 | Superficies de lectura, Settings y eliminaciones | Implementación aceptada el 2026-09-14; la instrumentación Keystore/Room y la limpieza física pasaron dentro de la corrida completa de `AND-09`. La acción de borrado integral de Crush pasó de la ficha a la fila activa conforme a `RP-DEL-006`, reutilizando el flujo durable existente; 5/5 pruebas instrumentadas y 463/463 JVM verificadas. Esto no certifica el borrado integral. La base posterior de Archivados existe en Android, pero su aceptación funcional queda para `FSR-07` |
| 8 | Paridad visual, formularios, accesibilidad y localización | Implementación aceptada el 2026-09-14; los recorridos en AVD corrigieron nombres accesibles de `MDLTextField`, switch biométrico y código de Email Verification, y conectaron el contenido privado de Experience en el Timeline del perfil. Los dos mensajes Vault de Experience usan recursos ES/EN y sus cuatro pruebas Compose dirigidas pasaron en una AVD separada. El scroll de Experience Add/Edit permite alcanzar la satisfacción obligatoria a escala extrema: 2/2 pruebas instrumentadas y 463/463 JVM verificadas; esta aceptación es sólo de layout, no del borrado legacy. El título biométrico de Settings tiene ahora espacio reservado junto al switch y envuelve sin colisión a escala extrema; 4/4 pruebas instrumentadas y 463/463 JVM verificadas, sin cambio de biometría funcional. Las tarjetas estadísticas de Home ya no colapsan el subtítulo ni recortan el valor a font scale 1.3 y densidad 546; 2/2 pruebas instrumentadas y 463/463 JVM verificadas, aceptación sólo de layout. Quedan dispositivo físico, contraste y revisión humana del inglés; lectores de pantalla y texto ampliado no son pendientes por `RP-UX-006` |
| 9 | Matriz final de pruebas, dispositivos y gates de distribución | En progreso; el alcance automatizado anterior incluyó la prueba Compose dirigida del `HomeScreen` reconstruido y el ensamblado release. La suite JVM actual tiene 463/463 verde. `AND-GOOGLE-01` pasó 2/2 instrumentadas en Pixel_9a/API 37. El manifiesto `androidTest` declara `ComponentActivity` en el paquete de tests y el host Compose funcionó en una AVD separada: Claude reportó 31/31 casos verdes en nueve clases, incluidos 4/4 de Experience y 5/5 del control; el XML conservado entonces verificó el último caso 1/1. El XML actual verifica 5/5 casos de ubicación y confirmación de «Eliminar Crush»; las corridas anteriores verificaron 2/2 casos de Home, 4/4 de Settings y 2/2 de Experience a escala extrema. Quedan acciones inertes del Timeline del perfil, interoperabilidad física iOS↔Android, biometría física, contraste, revisión humana del inglés y revisión Google Play/Legal autorizada; lectores de pantalla y texto ampliado no son gates |

Las fases técnicas y sus criterios viven únicamente en el plan Android. Se implementa una fase por sesión y no se avanza automáticamente a la siguiente.

## Estado inicial constatado

Android ya tiene composición Hilt, sesión y bootstrap verificados, Vault criptográficamente compatible, motor durable, flujos no bloqueados de edición/borrado de Crush y Experience/Timeline, y las superficies funcionales de `AND-07` aceptadas técnicamente. `AND-08` proporciona targets táctiles reales de 48dp, una reserva síncrona que serializa el submit, `displayName` no-Relationship por recursos y clasificación neutral/localizada de errores visibles. En `AND-09` se validaron Login, Register y el destino autenticado/Home mediante el grafo Hilt productivo, incluida una prueba Compose del `HomeScreen` reconstruido. Add Crush usa los tres ejes nativos y persiste una intención con UUIDs y entries Vault para Crush, Relationship y Experience opcional; sus correcciones de revisión y de compatibilidad de snapshot están aceptadas con evidencia JVM reportada. Los dos mensajes Experience `Locked`/`Corrupt` están localizados y sus cuatro pruebas Compose dirigidas pasaron en la AVD separada; el host compartido quedó validado funcionalmente. El scroll de Experience Add/Edit permite alcanzar la satisfacción obligatoria a font scale y display size extremos, con 2/2 pruebas instrumentadas y 463/463 JVM verificadas; la prueba no valida el borrado, que `ExperienceEditScreen` sigue ofreciendo sin la elegibilidad de `RP-EXP-019..021`. El título biométrico de Settings reserva espacio antes del switch y envuelve sin colisión, con 4/4 pruebas instrumentadas y 463/463 JVM verificadas; no se alteró la biometría funcional. Las tarjetas estadísticas de Home ya no colapsan el subtítulo a altura cero ni recortan el valor a escala extrema: `IntrinsicSize.Min` sustituye la altura fija, con 2/2 pruebas instrumentadas y 463/463 JVM verificadas; la captura real y el fallo antes del fix fueron reportados por el implementador, no conservados como artefactos. La acción «Eliminar Crush» está sólo en la fila activa y ya no en la ficha, con el mismo disparador durable y 5/5 instrumentadas verificadas; Android conserva una base visible de Archivados con consulta y restauración, pero su contrato completo de búsqueda y filtros aún no cumple `RP-LIS-001..004`; queda intacta y su aceptación funcional se revisa en `FSR-07`. El menú del Timeline del perfil continúa inerte y sus condiciones legacy tampoco están aceptadas. Siguen pendientes biometría física, contraste, interoperabilidad, revisión humana del inglés y distribución; lectores de pantalla y texto ampliado quedan fuera por `RP-UX-006`. El usuario confirmó el 2026-09-11 que la migración inicial sin commit hacia Hilt/repositorios era el baseline intencional: se preserva y se continúa desde allí, sin resetearla ni reemplazarla por una copia nueva de iOS.

Esta constatación es una entrada del plan, no una auditoría histórica ni una aceptación del código actual.

## Criterios de aceptación final

- Todas las dimensiones del contrato de paridad tienen evidencia Android y no conservan excepciones implícitas.
- GWT-AND-01 a GWT-AND-07 se validan en Android; GWT-AND-04 se valida en ambos sentidos con iOS.
- Ningún DTO funcional Android acepta o espera plaintext privado retirado del contrato.
- Vault, escrituras y borrados sobreviven offline, relanzamiento y respuestas ambiguas sin pérdida, duplicación ni falso éxito.
- Fotos, tokens, claves y datos privados quedan fuera de backups y logs no autorizados.
- La localización ES/EN y accesibilidad cubren todas las superficies de beta.
- Los contratos relacionales vigentes se implementan una sola vez contra [Relaciones](../../../rules/RELATIONSHIPS.md), [Experiencias](../../../rules/EXPERIENCES.md) y [Timeline](../../../TIMELINE.md), sin reintroducir el modelo anterior.
- Los requisitos legales y de Google Play necesarios para distribuir Android reciben revisión humana antes del rollout.
- El estado técnico estable se integra en documentos canónicos y ambos planes `android-parity` se eliminan al cerrar; Git conserva la historia.

## Decisiones abiertas

1. **Proveedores por plataforma — definido:** conforme a RP-LAN-004, RP-APL-034 y RP-GGL-002, Android ofrece email/contraseña y Google, pero no Apple; iOS ofrece email/contraseña y Apple, pero no Google. La implementación Google se ejecuta en su [plan propio](../sign-in-with-google/README.md) y nunca crea otro `User` ni Vault.
2. **Empty states:** `empty-state-standardization` está diferida post-beta también como referencia para Android. `AND-08` sólo debe impedir pantallas completamente vacías, confusión con error/loading o callejones sin salida; no adelanta el componente común ni la auditoría de once superficies prevista para después de la beta.
3. **Relationship:** Android adoptará el catálogo final de diez `RelationshipSituation`, incluido `3 FRIENDS` («Solo somos amigos»), períodos, hitos y capacidades latest-only, sin implementar el modelo legacy como solución transitoria.
4. **Distribución y Legal:** las obligaciones específicas de Android/Google Play y backups requieren alcance legal explícito; este plan no modifica `docs/legal/`.

## Próximo resultado permitido

`AND-GOOGLE-01` quedó aceptada con 463/463 pruebas JVM y 2/2 instrumentadas en el AVD Pixel_9a/API 37; `AND-GOOGLE-02` sigue bloqueada por el contrato API. La línea `android-parity` continúa `AND-09`: el manifiesto del host Compose y la localización Experience quedan aceptados tras la validación en la AVD separada `MDL_AND09_CLEAN_API37`, con rutas de discos propias verificadas antes de instalar la app. Claude reportó 31/31 casos verdes en nueve clases; el XML conservado entonces confirmó el último 1/1. El scroll de Experience Add/Edit y la separación visual del título biométrico de Settings están aceptados como correcciones de layout; la corrida de Settings verificó 4/4 casos y la suite JVM vigente 463/463. La pestaña Timeline de Crush Profile conserva un hallazgo cosmético que necesita decisión visual si se corrige. Las tarjetas estadísticas de Home quedan aceptadas como corrección de layout: la corrida de Home verificó 2/2 instrumentadas y la suite JVM vigente 463/463; dispositivo físico y contraste permanecen pendientes. La ubicación de «Eliminar Crush» conforme a `RP-DEL-006` queda aceptada como implementación de acceso: el XML actual verifica 5/5 instrumentadas y 463/463 JVM, sin cambiar el motor durable; las validaciones físicas/humanas que no sean de lectores de pantalla o texto ampliado siguen pendientes. `RP-ARC-001` y `RP-LIS-001..004` ya fijan el contrato de Archivados. La base Android existente se conserva sin cambios y no se amplía ahora; la revisión de búsqueda, filtros, navegación y restauración contra ese contrato, incluida la conexión con API real, queda para el último bloque `FSR-07` de `frontend-screen-rules`. No es el siguiente trabajo de Android. Hasta entonces, Android se contrasta contra la funcionalidad definida desde el inventario iOS y mantiene diferencias sólo cuando sean convenciones nativas sin pérdida funcional. El menú y el borrado legacy de Experience esperan el contrato relacional final. Después siguen las validaciones físicas/externas restantes. La revisión Google Play/Legal y cualquier firma o publicación requieren autorización propia. La estandarización integral de empty states permanece diferida post-beta.

## Plan local

- [`android/docs/plans/active/android-parity/README.md`](../../../../../android/docs/plans/active/android-parity/README.md)

## Referencias

- [`PRODUCT.md`](../../../PRODUCT.md)
- [`ARCHITECTURE.md`](../../../ARCHITECTURE.md)
- [`VAULT.md`](../../../VAULT.md)
- [`TIMELINE.md`](../../../TIMELINE.md)
- [`STATISTICS.md`](../../../STATISTICS.md)
- [`vault-e2ee`](../vault-e2ee/README.md)
- [Guardado y sincronización](../../../rules/SYNC.md) y [arquitectura transversal](../../../ARCHITECTURE.md#5-flujo-de-escritura-durable)
- [Verificación de email](../../../rules/ACCOUNT.md#verificacion-email)
- [`form-ux-performance`](../form-ux-performance/README.md)
- [Relaciones](../../../rules/RELATIONSHIPS.md) y [Timeline](../../../TIMELINE.md)
- [`security-hardening`](../security-hardening/README.md)
- [`sign-in-with-google`](../sign-in-with-google/README.md)
