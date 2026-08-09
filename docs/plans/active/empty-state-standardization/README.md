# Estandarización de empty states en iOS

> **Estado:** pendiente de implementación
> **Proyecto:** iOS
> **Última actualización:** 2026-07-18

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

1. Crear el componente y las cadenas localizadas.
2. Integrarlo en lista/timelines, mapa/estadísticas y perfil; estos grupos pueden avanzar por separado.
3. Limpiar strings anteriores, actualizar documentación estable y validar accesibilidad/layout.

## Criterios de aceptación

- [ ] Las once variantes usan el componente común en las superficies activas.
- [ ] Cada CTA ejecuta la acción contextual correcta.
- [ ] Error, loading, vacío y contenido son mutuamente coherentes.
- [ ] Los textos existen en español e inglés y no están hardcodeados.
- [ ] VoiceOver y Dynamic Type funcionan sin pérdida de acciones o contenido.
- [ ] El grid de redes mantiene enlaces e indicadores actuales y reduce la altura de la tarjeta.
- [ ] No se incorpora trabajo sobre pantallas sin callers.

## Estado resumido

| Proyecto | Estado | Próximo resultado |
|---|---|---|
| iOS | Pendiente | IOS-01 — componente base y localización |

## Plan local

- iOS: `ios/docs/plans/active/empty-state-standardization/README.md`

## Referencias estables

- Sistema UI: `ios/docs/architecture/ui-system.md`
- Timeline: `ios/docs/domains/features/timeline-behavior.md`
- Estadísticas: `ios/docs/domains/features/statistics.md`
