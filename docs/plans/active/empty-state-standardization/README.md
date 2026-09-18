# Estandarización de empty states en iOS

> **Estado:** diferida post-beta. La beta está prevista para octubre de 2026 y esta iniciativa se
> retomará después de esa salida, con objetivo de completarla antes del lanzamiento de enero de
> 2027. La app ya contiene algunos estados vacíos puntuales, pero todavía no existe el componente
> común ni está auditada la cobertura de las once superficies. No bloquea la beta salvo que la
> validación física detecte una pantalla completamente vacía, un estado confundido con error/loading
> o un callejón sin salida para crear el primer dato.
> **Proyecto:** iOS
> **Ventana prevista:** después de la beta de octubre de 2026 y antes del lanzamiento de enero de 2027

> **Hallazgo físico pendiente de decisión pre-beta:** al abrir la lista de Crushes sin conexión, la
> carga remota vacía el contenido en memoria y presenta un alert modal genérico. No es un empty state
> válido. Si se corrige antes de beta, debe ser un correctivo acotado de error offline; si se difiere,
> queda incorporado a IOS-02 y no se mezcla con Relationship.

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

## Interacción entre proyectos

| Proyecto | Responsabilidad funcional |
|---|---|
| iOS | Renderizar estados, localizar copy, conectar CTAs, conservar errores/loading y validar accesibilidad |
| API | Sin cambios |

## Orden general

La ejecución queda pausada hasta después de la beta de octubre de 2026. Al retomarla, primero se
auditan los estados puntuales ya existentes para decidir cuáles se reutilizan, cuáles se sustituyen y
qué superficies siguen realmente sin tratamiento; no se parte de la premisa de que toda la app está
vacía de soluciones actuales.

1. Crear el componente y las cadenas localizadas.
2. Integrarlo en lista/timelines, mapa/estadísticas y perfil; estos grupos pueden avanzar por separado.
3. Limpiar strings anteriores, actualizar documentación estable y validar accesibilidad/layout.

## Criterios de aceptación

- [ ] Las once variantes usan el componente común en las superficies activas.
- [ ] Cada CTA ejecuta la acción contextual correcta.
- [ ] Error, loading, vacío y contenido son mutuamente coherentes.
- [ ] La lista conserva contenido previamente cargado ante un fallo de refresh y usa error inline en el primer load offline.
- [ ] Los textos existen en español e inglés y no están hardcodeados.
- [ ] VoiceOver y Dynamic Type funcionan sin pérdida de acciones o contenido.
- [ ] El grid de redes mantiene enlaces e indicadores actuales y reduce la altura de la tarjeta.
- [ ] No se incorpora trabajo sobre pantallas sin callers.

## Estado resumido

| Proyecto | Estado | Próximo resultado |
|---|---|---|
| iOS | Diferida post-beta | Tras octubre de 2026: auditar los estados existentes y ejecutar IOS-01; completar la iniciativa antes del lanzamiento de enero de 2027 |

## Plan local

- iOS: `ios/docs/plans/active/empty-state-standardization/README.md`

## Referencias estables

- Arquitectura y sistema UI de iOS: [`ios/docs/ARCHITECTURE.md`](../../../../../ios/docs/ARCHITECTURE.md)
- Reglas canónicas de Timeline: [`product/docs/TIMELINE.md`](../../../TIMELINE.md)
- Reglas canónicas de estadísticas: [`product/docs/STATISTICS.md`](../../../STATISTICS.md)
