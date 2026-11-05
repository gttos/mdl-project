# Pendientes para revisar después

Lista única de hallazgos fuera del alcance activo, incorporados sólo con autorización del usuario conforme a [AGENTS.md](../../AGENTS.md#foco-y-hallazgos-fuera-de-alcance). Una entrada breve por hallazgo; sin planes ni historial. Registrar un pendiente no autoriza implementarlo ni cambia la tarea en curso.

- **iOS — filtros de Crushes en inglés:** revisar «Filtros», «Limpiar filtros» y «Género», que permanecen en español en la referencia fija `3977289e`. Hallazgo visual comprobado en el [par de filtros EN](../../outputs/and-par-06-visual/pairs/02_lista_filtros_en.png).
- **Android — Atrás desde la ficha:** revisar el comportamiento reportado en el que Atrás del sistema desde la ficha sale de la app, en lugar de volver a la lista. Referencias: `CrushListScreen` / `CrushProfileScreen`; sin verificación independiente de coordinación.
- **Android — regreso tras desbloquear:** revisar el comportamiento reportado en el que desbloquear desde la galería devuelve a la lista de Crushes. Referencias: `AppNavHost` / `CrushProfileScreen`; sin verificación independiente de coordinación. Registrar no autoriza corregirlo ni cambia AND-PAR-08.
