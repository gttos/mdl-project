# Estandarización de empty states en iOS

> **Estado:** correctivo pre-beta de estadísticas aceptado; implementación del correctivo offline de
> la lista aceptada y validada físicamente; correctivo de arranque offline aceptado y
> validado físicamente; estandarización completa diferida post-beta. El
> componente común y la auditoría de las once superficies se retomarán después de la beta de octubre
> de 2026, con objetivo de completarlas antes del lanzamiento de enero de 2027.
> **Proyecto:** iOS
> **Ventana prevista:** después de la beta de octubre de 2026 y antes del lanzamiento de enero de 2027

> **Decisión pre-beta vigente:** el fallo offline de la lista de Crushes se corrige antes de
> TestFlight como un correctivo acotado. Un primer load sin contenido confirmado muestra un error
> inline con Reintentar; un refresh fallido conserva el último estado confirmado —incluido un vacío
> confirmado— y usa feedback no bloqueante. Ninguno presenta un vacío nuevo ni un alert modal de
> carga. La estandarización visual de las once
> superficies continúa diferida y este correctivo no adelanta IOS-01–05.

> **Decisión pre-beta vigente:** al restaurar una sesión existente, no poder consultar por red el
> estado de verificación del email no convierte la cuenta en no verificada ni elimina la sesión. La
> app permanece cerrada a contenido privado y muestra un error de conexión con Reintentar. Un estado
> confirmado como no verificado abre el gate; uno verificado continúa el bootstrap; una respuesta de
> sesión inválida cierra la sesión. La beta no introduce un estado de verificación cacheado.

> **Decisión pre-beta vigente:** Daily Hacks conserva su carrusel y la marca «Próximamente»; no forma
> parte de este correctivo. Home no muestra cifras de maqueta y, cuando una respuesta exitosa no
> contiene estadísticas, ofrece «Agregar Crush». El Dashboard presenta el mismo vacío accionable en
> lugar de secciones sin datos. Esta entrega no declara completada la iniciativa de once superficies.

## Objetivo

Reemplazar los estados vacíos ad hoc de las superficies activas de iOS por un componente visual coherente, localizado, accesible y con una acción contextual cuando el usuario pueda resolver el vacío.

## Alcance

- Lista de crushes, diferenciando cuenta vacía y filtros sin resultados.
- Timeline global, timeline de perfil y actividad reciente.
- Mapa y estadísticas.
- Redes sociales, fecha de nacimiento, profesión y características del perfil.
- Rediseño compacto de redes sociales cuando existen datos.

Fuera de alcance:

- Pantallas o componentes que no están conectados a la navegación de lanzamiento.
- Nuevos flujos de Vault.
- Cambios de API, modelos de dominio o reglas de negocio.
- El estado textual del header del perfil, que no es una superficie vacía independiente.

## Comportamiento esperado

Todos los estados usan `MDLEmptyStateView` con título, mensaje, icono y CTA opcional. Los errores de red continúan usando el componente de error y nunca aparecen al mismo tiempo que un empty state.

En la lista, un fallo de refresh con contenido ya cargado conserva ese contenido y muestra feedback
offline no bloqueante. Un primer load sin datos confirmados muestra un error inline con Reintentar;
nunca un alert modal simultáneo con un vacío ni un mensaje que trate la falta de API como cuenta sin
Crushes.

| ID | Contexto | Acción principal |
|---|---|---|
| ES-01 | Lista sin crushes | Agregar crush |
| ES-02 | Búsqueda o filtros sin coincidencias | Limpiar filtros |
| ES-03 | Timeline global vacío | Agregar crush |
| ES-04 | Timeline del perfil vacío | Agregar experiencia |
| ES-05 | Actividad reciente vacía | Ir al timeline |
| ES-06 | Mapa sin ubicaciones | Ver crushes |
| ES-07 | Estadísticas sin datos | Agregar crush |
| ES-08 | Perfil sin redes sociales | Agregar redes |
| ES-09 | Perfil sin fecha de nacimiento | Agregar fecha |
| ES-10 | Perfil sin profesión | Agregar profesión |
| ES-11 | Perfil sin características | Agregar características |

El estado de estadísticas complementa el skeleton existente después de una carga exitosa sin datos; no lo sustituye durante loading. Las tarjetas de perfil usan la variante compacta. Las redes existentes se muestran en un grid adaptable de hasta dos columnas.

Para ES-07, `moment.status = empty` no basta para declarar vacío el Dashboard: una cuenta sin Crushes
activos puede conservar métricas históricas de Crushes archivados. El vacío total sólo existe cuando
la respuesta exitosa de API no aporta evidencia estadística en ninguna dimensión: las métricas
opcionales del resumen son nulas; las colecciones de match no contienen ningún recuento positivo
(pueden venir vacías o con las filas canónicas en cero); y todas las secciones variables están en
estado vacío sin valor ni muestra usada. Un recuento negativo o una sección declarada vacía que
incluya valor o muestra positiva se trata como una incoherencia y falla cerrado, sin ocultar el
Dashboard. `summary.actualStatus = single` por sí solo no constituye evidencia. Loading, primer error
y recarga fallida siguen siendo estados distintos; una recarga fallida conserva el último contenido
confirmado y comunica el error sin presentarlo como vacío.

## Interacción entre proyectos

| Proyecto | Responsabilidad funcional |
|---|---|
| iOS | Renderizar estados, localizar copy, conectar CTAs, conservar errores/loading y validar accesibilidad |
| API | Sin cambios |

## Orden general

Antes de TestFlight se ejecutan el correctivo de ES-07 en Home y Dashboard, el correctivo offline
acotado de la lista de Crushes y la separación entre error de red y email no verificado durante la
restauración de sesión. Después de la beta se auditan los estados puntuales ya existentes para
decidir cuáles se reutilizan, cuáles se sustituyen y qué superficies siguen realmente sin
tratamiento; no se parte de la premisa de que toda la app está vacía de soluciones actuales.

1. Crear el componente y las cadenas localizadas.
2. Integrarlo en lista/timelines, mapa/estadísticas y perfil; estos grupos pueden avanzar por separado.
3. Limpiar strings anteriores, actualizar documentación estable y validar accesibilidad/layout.

## Criterios de aceptación

- [ ] Las once variantes usan el componente común en las superficies activas.
- [ ] Cada CTA ejecuta la acción contextual correcta.
- [ ] Error, loading, vacío y contenido son mutuamente coherentes.
- [ ] La lista conserva contenido previamente cargado ante un fallo de refresh y usa error inline en el primer load offline.
- [ ] Los textos existen en español e inglés y no están hardcodeados.
- [ ] ES/EN conservan acciones y contenido con el tamaño de texto normal; lectores de pantalla y texto ampliado quedan fuera por `RP-UX-006`.
- [ ] El grid de redes mantiene enlaces e indicadores actuales y reduce la altura de la tarjeta.
- [ ] No se incorpora trabajo sobre pantallas sin callers.

## Estado resumido

| Proyecto | Estado | Próximo resultado |
|---|---|---|
| iOS | Correctivos pre-beta IOS-00, IOS-00L e IOS-00B aceptados y validados | Después de la beta, auditar y ejecutar IOS-01–05 |

## Plan local

- iOS: `ios/docs/plans/active/empty-state-standardization/README.md`

## Referencias estables

- Arquitectura y sistema UI de iOS: [`ios/docs/ARCHITECTURE.md`](../../../../../ios/docs/ARCHITECTURE.md)
- Reglas canónicas de Timeline: [`product/docs/TIMELINE.md`](../../../TIMELINE.md)
- Reglas canónicas de estadísticas: [`product/docs/STATISTICS.md`](../../../STATISTICS.md)
