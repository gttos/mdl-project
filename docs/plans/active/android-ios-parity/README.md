# Paridad Android con iOS

<a id="objetivo-y-alcance"></a><a id="referencia-fija-y-evolución-de-ios"></a><a id="relevamiento-y-diferencias-de-partida"></a><a id="método-acordado-y-criterios-de-avance"></a><a id="entrega-a-claude-y-revisión-de-resultados"></a><a id="orden-y-estado-de-entrega"></a><a id="dependencias-y-límites-de-aceptación"></a><a id="cierre-y-siguiente-fase"></a>

> **Estado:** AND-PAR-01..11 aceptadas como implementación con validaciones pendientes. AND-PAR-12 recibida y no aceptada: hay implementación y compilación reportadas, con correctivos de aislamiento de cuenta, reanudación de baja y comparación visual pendientes en su alcance independiente. Google conserva sus dependencias propias; no es la única parte que falta de 12. AND-PAR-13 no se inicia. Las excepciones visuales expresas de recuperación (03) y mapa real (10, incluida su clave) deben resolverse antes de aceptar paridad final o distribuir. Tests, fallos/carreras, integración completa y físico permanecen en 14/15.

## Alcance y referencia

Reproducir los recorridos, datos, acciones y garantías del corte iOS `3977289e8f06571c47ef2aa697513c2078d68c7b`. Los incrementos posteriores se reconcilian en AND-PAR-13 antes de validar y distribuir. Android conserva su distribución independiente de la beta iOS/API.

- [Objetivo de distribución](../../../FUNCIONES_FUTURAS_PARA_JIRA.md#android).
- [Plan local y criterios de presentación](../../../../../android/docs/plans/active/android-ios-parity/README.md).
- [Google](../sign-in-with-google/README.md).

## Método y aceptación

La implementación completa precede a adaptación y ejecución de tests automatizados; durante implementación se leen consumidores y fixtures y se compila producción. Cada fase conserva su comparación de presentación con el corte fijo conforme al plan local. Las excepciones de recuperación y mapa permanecen diferidas y deben resolverse antes de aceptación final o distribución. La coordinación contrasta evidencia y actualiza el estado vigente sin ejecutar suites.

## Estado y orden

| Fase local | Resultado | Estado / dependencia |
|---|---|---|
| AND-PAR-01 | Formularios compartidos, navegación y entorno local | Implementación aceptada con validaciones pendientes; comparación visual ES/EN del corte revisada. Tests, Autofill con proveedor real e integración física final en 14/15. |
| AND-PAR-02 | Email/contraseña, capacidad de registro, OTP, recuperación de acceso y lectura Legal | Implementación aceptada con validaciones pendientes; salida visual ES/EN del corte cerrada. Tests e integración/validación física final en 14/15. |
| AND-PAR-03 | Vault interoperable, encapsulaciones, onboarding y recovery | Implementación aceptada con validaciones pendientes. Comparación visual de recuperación/24 palabras diferida expresamente por el usuario, sin bloquear 04. Tests/interoperabilidad en 14/15. |
| AND-PAR-04 | Bootstrap exclusivo, App Lock y login biométrico por desafío | Implementación aceptada con validaciones pendientes. Comparación visual del corte aceptada con límites; sin compensación remota especulativa. Tests/dobles, concurrencia, firma API y físico en 14/15. |
| AND-PAR-05 | Contrato relacional base y motor durable final, POST Vault, proyecciones, conflictos, corrección y borrados | Implementación aceptada con validaciones pendientes. Comparación visual ES/EN aceptada. Tests/dobles, concurrencia, integración/interoperabilidad y físico en 14/15; cuota de Ajustes en 12. |
| AND-PAR-06 | Crush: alta, listas activas/archivadas, búsqueda/filtros, ficha y edición completa | Implementación aceptada con validaciones pendientes. Comparación visual ES/EN aceptada con estados reales/preparados; no acredita integración completa. Tests/dobles, integración completa y físico en 14/15; Fotos/Timeline de ficha en 08/07 y deltas posteriores al corte sujetos a adopción explícita. |
| AND-PAR-07 | Experiencias, hitos/períodos, Timeline y acciones latest-only | Implementación aceptada con validaciones pendientes. Comparación ES/EN aceptada con estados reales/preparados; no acredita integración completa. Tests/dobles, rechazo real de API, integración completa, interoperabilidad y físico en 14/15; incrementos futuros en 13. |
| AND-PAR-08 | Fotos locales cifradas y galería/visor | Implementación aceptada con validaciones pendientes. Comparación ES/EN aceptada con diferencias nativas y datos sintéticos; no acredita integración completa. Tests, fallos/carreras, integración completa, HEIC y validación física en 14/15. |
| AND-PAR-09 | Home y navegación integrada de actividad/consejos/vacíos | Implementación aceptada con validaciones pendientes. Comparación ES/EN aceptada con estados preparados; no acredita integración completa. Tests/dobles, rutas de fallo/carrera, integración completa y físico en 14/15; dashboard de Home en 11. |
| AND-PAR-10 | Mapa y conservación del contexto de lista | Implementación aceptada con validaciones pendientes y excepción visual expresa del usuario. Clave Google Maps y evidencia cartográfica real ES/EN diferidas, sin bloquear 11. La base sintética sólo acredita componentes. No se declara mapa real validado; pendiente antes de aceptar paridad final o distribuir. |
| AND-PAR-11 | Dashboard completo de estadísticas | Implementación aceptada con validaciones pendientes. Comparación ES/EN aceptada para dashboard, Home y avisos; los estados preparados no acreditan integración completa. Tests/dobles, fallos/carreras, integración completa y físico en 14/15. |
| AND-PAR-12 | Ajustes, credenciales, baja, tutoriales y demo nativa completa | No aceptada. Alcance independiente parcialmente implementado/compilado; quedan aislamiento de las acciones y sus resultados, conservación del intento de baja y evidencia/correctivos de presentación del plan local. Integra Google sin absorber su lifecycle: API/AND-GOOGLE-03 siguen pendientes; no inventar contratos ni habilitar el proveedor. |
| AND-PAR-13 | Reconciliación del corte iOS/API final y de funciones incorporadas después del SHA inicial | Pendiente; 12; nuevo SHA y alcance explícitos antes de implementar deltas. |
| AND-PAR-14 | Portado/adaptación y ejecución de pruebas JVM, contratos, cripto y modelos/flujo | Pendiente; toda la implementación acordada terminada. |
| AND-PAR-15 | Instrumentación Compose, integración local, regresión visual conjunta y validación física/cruzada | Pendiente; 14; no sustituye la comparación visual exigida durante cada fase. |
| AND-PAR-16 | Readiness del lanzamiento conjunto y cierre documental coordinado | Pendiente; 15, Google y gates públicos aplicables resueltos. |

## Escenarios compartidos de aceptación

Los escenarios se aplican al corte fijo y se reconcilian en AND-PAR-13; no adelantan las validaciones diferidas ni levantan las excepciones expresas del estado vigente.

| ID | Dado / Cuando / Entonces | Reglas |
|---|---|---|
| GWT-PAR-01 | Dado un recorrido equivalente y el mismo idioma/datos, al ejecutarlo en ambos clientes se conservan pasos, acciones, validación, mensajes y resultados conforme al catálogo y al corte aprobado. | RP-BASE-008, RP-LAN-008, RP-FOR-001..014. |
| GWT-PAR-02 | Dadas entrada, registro pendiente, sesión válida/inválida y Vault nuevo/remoto/bloqueado, al entrar o relanzar sólo aparece el destino autorizado, sin frames privados previos. | Cuenta, RP-CUE-003/010..016, BR-3.12..15, RP-VLT-011..018/032..035. |
| GWT-PAR-03 | Dada la misma acción válida o inválida, al guardar y reintentar con fallo de red, rechazo o relanzamiento se conservan identidad, fechas, intención y estados; la inválida no produce efectos. | RP-ID-001..008, RP-AGE-001..015, RP-DUR-001..011, RP-RET-001..006, RP-RES-001..010. |
| GWT-PAR-04 | Dada una historia con bloque inicial, Experiences independientes/vinculadas, cierre y reapertura, al consultar, filtrar, corregir o borrar se obtienen las mismas entidades, agrupación y capacidades. | RP-CRU-004, Relaciones, Experiencias, RP-TIM-001..029. |
| GWT-PAR-05 | Dados Vaults sintéticos creados por cada plataforma, al recuperar en la otra mediante cada mecanismo se obtiene el mismo diario, sin restaurar fotos locales ni permitir dos copias activas fuera del contrato. | RP-VLT-013..018/026/032..034, GWT-LPE-10. |
| GWT-PAR-06 | Dadas fotos válidas/inválidas y cambios de sesión/bloqueo, al importar, mostrar, retirar o borrar se cumplen los escenarios canónicos sin persistencia legible ni publicación tardía. | GWT-LPE-01..12, RP-LOCK-003, RP-ID-007. |
| GWT-PAR-07 | Dadas respuestas equivalentes de Home/mapa/dashboard y estados de lectura, al navegar/actualizar se conserva el contenido confirmado, sus proyecciones y las cifras publicadas sin cálculo cliente alternativo. | RP-HOM-001..007, RP-MAP-001..005, RP-EST-001..027, RP-VAC-001..004. |
| GWT-PAR-08 | Dado el equivalente Android de demo/tutoriales, al explorar y volver se reproducen destinos, contenido literal y salidas de la etapa aprobada sin tocar sesión, diario o Vault reales. | GWT-ODM-S01..S13 y demás GWT aplicables de UX; RP-LAN-008. |
| GWT-PAR-09 | Dadas cuentas A/B, bajas confirmadas/ambiguas y cambios de credenciales, al continuar o relanzar no se mezclan claves, proyecciones, fotos o intenciones ni se anticipa limpieza. | RP-ID-007, RP-BAJ-001..012, RP-OTP-007, GWT-GOOGLE-08/09. |
| GWT-PAR-10 | Dadas pantallas equivalentes ES/EN con tamaño de texto normal, al compararlas se conservan jerarquía visual, recursos, campos, gráficos, estados y navegación usando controles nativos de cada plataforma. | RP-LAN-008, RP-UX-006, RP-DEMO-003/014. |

## Próxima fase

Continuar únicamente los correctivos de AND-PAR-12 del plan local: acciones ligadas a su cuenta/sesión de origen, conservación y resolución del mismo intento de baja y comparación visual completa del alcance independiente. La evidencia de Ajustes/cuenta/credenciales/baja/ayudas y tutoriales se completa ahora, sin trasladarla a 15. Mantener las dependencias de Google y la excepción de cartografía real/clave, sin activar analítica. Las diferencias de presentación propias de 12 se corrigen dentro de la fase. Mapear consumidores y pruebas, compilar sólo producción y conservar los tests existentes sin modificarlos ni ejecutarlos. AND-PAR-13 no se inicia; no se modifican API/iOS/Legal ni definiciones funcionales de Product. Las validaciones automatizadas, fallos/carreras, integración completa, interoperabilidad y pruebas físicas permanecen en 14/15.
