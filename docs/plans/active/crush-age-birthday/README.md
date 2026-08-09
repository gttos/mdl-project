# Edad y cumpleaños de un crush

> **Estado:** activo; decisión de producto aprobada
> **Proyectos:** API e iOS
> **Última actualización:** 2026-07-21

## Objetivo

Mantener el editor actual de seis partes y permitir que el usuario introduzca datos fáciles de conocer —edad actual y día/mes de cumpleaños— sin pedir un año de nacimiento, sin recalcular desde `metAt` y sin volver a guardar una edad que el usuario no tocó.

## Alcance

- Incluye el significado de `age`, una fecha de confirmación `age_as_of`, el cálculo local de edad actual y la edición de día/mes.
- Incluye la creación y edición de crushes con el editor existente, sin pasos ni botones adicionales.
- Incluye la corrección del mapeo de edad/cumpleaños en la migración legacy y la reconstrucción completa del único usuario real desde la fuente legacy.
- Incluye la convivencia temporal con clientes anteriores durante el despliegue coordinado.
- Fuera de alcance: reparar manualmente filas ya migradas, conservar años sintéticos, derivar edad desde `metAt` o enviar día/mes de cumpleaños a la API.
- Fuera de alcance de esta entrega: implementar recordatorios de cumpleaños. Cuando se aborden, serán notificaciones locales de iOS porque la API no conoce el cumpleaños ni el nombre privado.

## Comportamiento esperado

### Significado de los datos

- `age` es la última edad actual confirmada por el usuario.
- `age_as_of` es el día civil en que esa edad fue confirmada.
- El cumpleaños es únicamente día y mes, es opcional y pertenece al Vault local E2EE.
- No se persiste ni se presenta un año de nacimiento que el usuario no haya introducido.
- `metAt` describe cuándo conoció al crush y no participa en ningún cálculo de edad.

### Alta

1. En la primera parte del editor, el usuario introduce la edad actual.
2. En la segunda parte, puede introducir día y mes de cumpleaños.
3. iOS envía `age` y `age_as_of` a la API; `age_as_of` es la fecha local del día del alta.
4. iOS guarda día/mes en el Vault. Solo el `zodiac_id` derivado puede enviarse a la API.

### Edición

- Se conserva un único editor de seis partes. No se añade “Editar edad”, confirmación extra ni otro flujo.
- La primera parte muestra la edad actual calculada cuando hay día/mes y ancla; si no hay cumpleaños, muestra la última edad confirmada.
- Cargar el formulario o actualizar un valor por cálculo interno no marca la edad como editada.
- Si el usuario no toca la edad, el PATCH omite tanto `age` como `age_as_of`; cualquier otro cambio conserva ambos valores en API.
- Si el usuario toca la edad, el PATCH envía el nuevo `age` y `age_as_of` con la fecha local actual.
- Si cambia día/mes sin tocar la edad, iOS conserva la pareja `age`/`age_as_of` y vuelve a calcular la edad visible con el cumpleaños nuevo.
- Si cambia día/mes, iOS actualiza una sola entry del crush en Vault y envía el `zodiac_id` derivado; si lo elimina, borra el cumpleaños local y envía `zodiac_id = null`.

### Cálculo local

- La edad visible es la edad confirmada más la cantidad de cumpleaños ocurridos después de `age_as_of` y hasta la fecha local actual.
- El cálculo no necesita almacenar un año de nacimiento.
- Para un cumpleaños el 29 de febrero, en años no bisiestos el aniversario de cálculo se considera el 28 de febrero.
- Sin día/mes no existe base para incrementar la edad con exactitud; se mantiene la última edad confirmada hasta que el usuario la cambie.

### Reconstrucción legacy

- La solución para los datos ya migrados es ejecutar otra vez una migración legacy corregida y completa; no se reparan crushes uno por uno.
- El migrador usa un modelo legacy dedicado que conserva los campos privados mientras sigan disponibles en la fuente. No reutiliza modelos de lectura anonimizados.
- Producción ya no conserva la columna legacy `birthdate`. Por decisión del usuario no se recuperarán cumpleaños desde backups locales ni desde el Vault anterior: durante la reconstrucción todos permanecen ausentes y se volverán a cargar manualmente después de verificar el conjunto nuevo. No se fabrica `01/01`, `31/12` ni otra fecha.
- La API prepara la pareja legacy conservando siempre el `age` registrado y usando la fecha civil de `created_at` como `age_as_of`. El año de `birthdate` no recalcula la edad porque no fue confirmado; únicamente día/mes pueden pasar después al Vault. `metAt` no interviene.
- El inventario del cumpleaños es independiente del backfill de edad: una fecha ausente, imposible o sospechosa se reporta para la reconstrucción Vault, pero no bloquea una pareja `age`/`age_as_of` válida.
- La preparación y la reconstrucción solo consideran crushes activos y no anonimizados; los registros eliminados o anonimizados quedan fuera del conjunto migrable.
- La reconstrucción crea un conjunto nuevo y versionado, verifica el inventario exacto y solo entonces lo activa. Las entries defectuosas anteriores quedan como versiones históricas no activas; no se mezclan con el conjunto reconstruido.
- Un error parcial no marca la migración como completa ni reemplaza el Vault activo.

## Interacción entre proyectos

| Proyecto | Responsabilidad funcional |
|---|---|
| API | Conservar `age` y `age_as_of` como pareja, omitir cambios no solicitados, devolver el snapshot confirmado y preparar el ancla de registros legacy |
| iOS | Mantener el editor actual, detectar interacción real con la edad, calcular la edad visible, custodiar día/mes en Vault y ejecutar la reconstrucción legacy corregida |

Los contratos funcionales normales de API no reciben día/mes, año de nacimiento, `isFromVault` ni otro indicador interno del Vault. La lectura legacy temporal queda limitada al migrador hasta el cleanup autorizado.

## Orden general

1. Cerrar la escritura de una única entry y el orden persistencia→refresco de iOS-09.1 en `vault-e2ee`.
2. Añadir y probar el contrato `age`/`age_as_of` en API.
3. Adaptar cálculo, alta y editor iOS sin modificar la estructura de seis partes.
4. Preparar el backfill legacy de edad con dry-run y aprobación de su reporte.
5. Completar envelope, aislamiento e inventario de migración exigidos por `vault-e2ee`.
6. Ejecutar desde el botón existente la reconstrucción legacy completa del usuario real y verificarla antes de activarla.
7. Continuar con cleanup legacy únicamente desde el plan `vault-e2ee` y con su evidencia correspondiente.

## Criterios de aceptación

- [ ] No existe ningún botón o paso adicional para editar edad.
- [ ] Guardar otra parte del editor sin tocar edad no cambia `age` ni `age_as_of`.
- [ ] Tocar edad actualiza ambos valores con una única acción de guardado.
- [ ] Cambiar día/mes recalcula la edad visible y el zodiaco sin persistir un año ni usar `metAt`.
- [ ] La API nunca recibe día/mes de cumpleaños ni flags internos del Vault.
- [ ] Editar un crush conserva exactamente una entry activa y Home/perfil refrescan después de persistirla.
- [ ] La migración corregida deja todos los cumpleaños ausentes, sin recuperar valores del Vault anterior ni fabricar fechas; el usuario podrá introducir día/mes manualmente después.
- [ ] La reconstrucción completa puede repetirse sin duplicar entidades activas y sin marcar éxito ante fallos parciales.
- [ ] El usuario real queda verificado por inventario exacto antes de activar el Vault reconstruido.

## Estado resumido

| Proyecto | Estado | Próximo resultado |
|---|---|---|
| API | API-AGE-01 y API-AGE-02 desplegadas; producción verificada con 183/183 preparados, 0 inválidos y 0 fallidos | Mantener compatibilidad hasta distribuir iOS; API-AGE-03 sigue bloqueada |
| iOS | iOS-09.1a completada manualmente; Home, navegación y lock/unlock funcionan. Escrituras iOS-09.1 aún pendientes de cierre | iOS-AGE-01 después de cerrar la validación de escrituras requerida |
| Transversal | Reconstrucción aprobada, todavía no ejecutada | Corregir primero el writer y el migrador; luego reconstruir, no reparar |

## Planes locales

- API: `api/docs/plans/active/crush-age-birthday/README.md`
- iOS: `ios/docs/plans/active/crush-age-birthday/README.md`

## Referencias estables

- Regla de producto: `product/docs/product/crush-age-and-birthday.md`
- Feature Vault: `product/docs/plans/active/vault-e2ee/README.md`
- Ciclo de vida iOS del crush: `ios/docs/domains/features/crush-lifecycle.md`
- Arquitectura: `product/docs/architecture/system-overview.md`
