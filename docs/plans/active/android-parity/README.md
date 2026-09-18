# Paridad Android

> **Estado:** activa; `AND-01` a `AND-04` aceptadas, y `AND-05`/alcance no bloqueado de `AND-06`/`AND-07`/`AND-08` con implementación aceptada; `AND-09` está en progreso, con su alcance automatizado y tres correcciones accesibles/visuales aceptadas con validaciones pendientes. El slice Add Crush está aceptado con validaciones pendientes: preserva snapshots previos, corrige revisiones Vault y cubre creación durable completa. También quedan la verificación dirigida de Home, acciones inertes del Timeline del perfil, localización residual, recorridos físicos/humanos y validaciones externas
> **Proyectos:** Product / Android; API e iOS como contratos y evidencia, no como ámbitos de implementación por defecto
> **Última actualización:** 2026-09-15

## Objetivo

Incorporar Android como cliente nativo de MDL con el mismo comportamiento observable, contrato HTTP, propiedad de datos, garantías de Vault, semántica durable y calidad de la beta iOS vigente, usando una implementación idiomática de Android.

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

**Dado** un alta, edición, lectura o borrado con nombre/contacto/redes/cumpleaños, lugar/descripción o nota de cierre,

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
| 6 | Flujos de dominio y Timeline | Implementación aceptada el 2026-09-13 para el alcance no bloqueado de edición/borrado de Crush y Experience/Timeline; migración Room v2→v3 y cola/atomicidad Vault ejecutadas en instrumentación dentro de `AND-09`. El slice Add Crush está aceptado con validaciones pendientes: selector nativo, bundle Vault durable, proyección/descarte, corrección con revisión nueva y lectura compatible de snapshots anteriores; evidencia JVM reportada 450/450 | Pendientes propios de `AND-09`, no de este slice: Home, acciones del Timeline del perfil, localización y validaciones físicas/externas |
| 7 | Superficies de lectura, Settings y eliminaciones | Implementación aceptada el 2026-09-14; la instrumentación Keystore/Room y la limpieza física pasaron dentro de la corrida completa de `AND-09` |
| 8 | Paridad visual, formularios, accesibilidad y localización | Implementación aceptada el 2026-09-14; los recorridos en AVD corrigieron nombres accesibles de `MDLTextField`, switch biométrico y código de Email Verification, y conectaron el contenido privado de Experience en el Timeline del perfil. Quedan dos mensajes Vault de Experience hardcoded en inglés, revisión humana con TalkBack, tamaños extremos pendientes y revisión humana del inglés |
| 9 | Matriz final de pruebas, dispositivos y gates de distribución | En progreso; alcance automatizado aceptado el 2026-09-14 con 385/385 JVM, 147/147 instrumentados y builds debug/release verdes en un único emulador Pixel 9a/API 37. Add Crush reporta 450/450 JVM y la revisión transversal aceptó su código; queda reconfirmar el ensamblado release de esta última pasada. También quedan acciones inertes del Timeline del perfil, prueba dirigida del `HomeScreen` reconstruido, interoperabilidad física iOS↔Android, biometría en dispositivo físico, TalkBack escuchado por una persona, superficies/tamaños restantes, revisión humana del inglés y revisión Google Play/Legal autorizada |

Las fases técnicas y sus criterios viven únicamente en el plan Android. Se implementa una fase por sesión y no se avanza automáticamente a la siguiente.

## Estado inicial constatado

Android ya tiene composición Hilt, sesión y bootstrap verificados, Vault criptográficamente compatible, motor durable, flujos no bloqueados de edición/borrado de Crush y Experience/Timeline, y las superficies funcionales de `AND-07` aceptadas técnicamente. `AND-08` proporciona targets táctiles reales de 48dp, una reserva síncrona que serializa el submit, `displayName` no-Relationship por recursos y clasificación neutral/localizada de errores visibles. En `AND-09` se corrigió la infraestructura, se ejecutó el gate automatizado completo y se validaron Login, Register y el destino autenticado/Home mediante el grafo Hilt productivo. Recorridos posteriores sobre el AVD Pixel 9a/API 37 atravesaron el gate con huella emulada, ampliaron la cobertura hasta Crush Profile, Experience add/edit y Email Verification, y corrigieron tres defectos accesibles/visuales. Add Crush usa los tres ejes nativos y persiste una intención con UUIDs y entries Vault para Crush, Relationship y Experience opcional; sus correcciones de revisión y de compatibilidad de snapshot están aceptadas con evidencia JVM reportada. La reconstrucción accidental de `HomeScreen`, el menú inerte del Timeline del perfil y dos mensajes Experience no localizados requieren corrección/verificación dirigida. Siguen pendientes la escucha humana de TalkBack, biometría física, tamaños extremos restantes, interoperabilidad, revisión humana del inglés y distribución. El usuario confirmó el 2026-09-11 que la migración inicial sin commit hacia Hilt/repositorios era el baseline intencional: se preserva y se continúa desde allí, sin resetearla ni reemplazarla por una copia nueva de iOS.

Esta constatación es una entrada del plan, no una auditoría histórica ni una aceptación del código actual.

## Criterios de aceptación final

- Todas las dimensiones del contrato de paridad tienen evidencia Android y no conservan excepciones implícitas.
- GWT-AND-01 a GWT-AND-07 se validan en Android; GWT-AND-04 se valida en ambos sentidos con iOS.
- Ningún DTO funcional Android acepta o espera plaintext privado retirado del contrato.
- Vault, escrituras y borrados sobreviven offline, relanzamiento y respuestas ambiguas sin pérdida, duplicación ni falso éxito.
- Fotos, tokens, claves y datos privados quedan fuera de backups y logs no autorizados.
- La localización ES/EN y accesibilidad cubren todas las superficies de beta.
- Los contratos de Relationship vigentes se implementan una sola vez después de estabilizar `relationship-redesign`.
- Los requisitos legales y de Google Play necesarios para distribuir Android reciben revisión humana antes del rollout.
- El estado técnico estable se integra en documentos canónicos y ambos planes `android-parity` se eliminan al cerrar; Git conserva la historia.

## Decisiones abiertas

1. **Sign in with Apple en Android:** el plan activo actual lo limita a API/iOS. Antes de la fase de proveedores de autenticación se debe decidir si Android lo expone o conserva sólo email/contraseña durante la beta.
2. **Empty states:** `empty-state-standardization` está diferida post-beta también como referencia para Android. `AND-08` sólo debe impedir pantallas completamente vacías, confusión con error/loading o callejones sin salida; no adelanta el componente común ni la auditoría de once superficies prevista para después de la beta.
3. **Relationship:** las decisiones abiertas y el contrato final pertenecen a `relationship-redesign`; Android no implementa el modelo legacy como solución transitoria.
4. **Distribución y Legal:** las obligaciones específicas de Android/Google Play y backups requieren alcance legal explícito; este plan no modifica `docs/legal/`.

## Próximo resultado permitido

Continuar `AND-09` mediante sesiones separadas para la prueba Compose del `HomeScreen` reconstruido, la localización residual y una salida no inerte del Timeline del perfil sin ampliar Relationship; luego retomar validaciones físicas o externas. Reconfirmar el ensamblado release antes del cierre de la matriz. La revisión Google Play/Legal y cualquier firma o publicación requieren autorización propia. La estandarización integral de empty states permanece diferida post-beta.

## Plan local

- [`android/docs/plans/active/android-parity/README.md`](../../../../../android/docs/plans/active/android-parity/README.md)

## Referencias

- [`PRODUCT.md`](../../../PRODUCT.md)
- [`ARCHITECTURE.md`](../../../ARCHITECTURE.md)
- [`VAULT.md`](../../../VAULT.md)
- [`TIMELINE.md`](../../../TIMELINE.md)
- [`STATISTICS.md`](../../../STATISTICS.md)
- [`vault-e2ee`](../vault-e2ee/README.md)
- [`durable-write-feedback`](../durable-write-feedback/README.md)
- [`email-verification`](../email-verification/README.md)
- [`form-ux-performance`](../form-ux-performance/README.md)
- [`relationship-redesign`](../relationship-redesign/README.md)
- [`security-hardening`](../security-hardening/README.md)
