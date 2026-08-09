# Edad y cumpleaños de un crush

> **Ámbito:** comportamiento compartido de producto entre iOS y API.
> **Fecha de decisión:** 2026-07-21.

## Datos que pide el producto

El usuario no debe calcular ni introducir un año de nacimiento. El editor de crushes conserva sus seis partes:

- La primera parte pide la edad actual.
- La segunda parte permite introducir día y mes de cumpleaños.
- No existe un botón, modal o paso separado para “editar edad”.

`metAt` conserva únicamente el significado de fecha en que el usuario conoció al crush. Nunca se usa para calcular edad o cumpleaños.

## Significado y propiedad

| Dato | Significado | Propietario |
|---|---|---|
| `age` | Última edad actual confirmada | API |
| `age_as_of` | Día civil en que se confirmó `age` | API |
| Día/mes de cumpleaños | Cumpleaños parcial y opcional | Vault E2EE en iOS |
| `zodiac_id` | Clasificación derivada de día/mes | API |

Los contratos funcionales nuevos no reciben día, mes, año de nacimiento ni flags internos como `isFromVault`. Los campos legacy pueden seguir temporalmente disponibles para el migrador dedicado hasta que `vault-e2ee` autorice su cleanup; no vuelven a escribirse desde el editor normal.

## Alta y edición

- Al crear un crush, iOS envía `age` junto con `age_as_of` igual a la fecha local actual.
- Al abrir el editor, la carga inicial no cuenta como una edición de edad.
- Si el usuario no toca el control de edad, guardar omite `age` y `age_as_of` y conserva el snapshot existente.
- Si lo toca, guardar actualiza ambos valores con una sola acción.
- Cambiar día/mes no reancla la edad. iOS actualiza el cumpleaños local, recalcula la presentación y envía únicamente el zodiaco derivado.
- Limpiar día/mes elimina el cumpleaños local y deja `zodiac_id` en null.

## Edad visible

Cuando existen cumpleaños y ancla, iOS suma a la última edad confirmada la cantidad de cumpleaños ocurridos después de `age_as_of` y hasta hoy. El cálculo no persiste un año de nacimiento. Para el 29 de febrero, el aniversario se considera el 28 de febrero durante un año no bisiesto.

Sin cumpleaños, la app muestra la última edad confirmada: no puede incrementarla de forma exacta sin conocer día y mes.

## Conversión de registros legacy

- El `age` legacy se conserva como la edad que el usuario confirmó al crear el registro.
- `age_as_of` se ancla en la fecha civil de `created_at`, que es la evidencia disponible de cuándo se introdujo ese valor.
- El año de `birthdate` legacy no se usa para recalcular `age`: fue derivado históricamente y no constituye una fecha de nacimiento confirmada.
- De `birthdate` solo pueden recuperarse día y mes para el migrador Vault. `metAt` no participa en esta conversión.
- Si faltan `created_at` o una edad válida, el registro se reporta como inválido y no se inventa un ancla.
- La preparación de `age_as_of` no depende de que el cumpleaños legacy exista o sea confiable. Los cumpleaños ausentes, imposibles o sospechosos se cuentan aparte para revisar la reconstrucción Vault, pero no alteran ni bloquean una pareja de edad válida.
- Los registros eliminados o anonimizados no forman parte del backfill ni de la reconstrucción Vault.
- Decisión operativa para el único usuario real: producción ya no conserva la columna `birthdate` y no se utilizarán backups locales ni el Vault anterior para intentar recuperarla. La reconstrucción deja el cumpleaños ausente; el usuario volverá a introducir manualmente día y mes después de verificar la migración completa.

## Recordatorios

El objetivo futuro del cumpleaños es permitir un aviso de retorno una semana antes y/o el mismo día. Ese recordatorio se programará localmente en iOS para no revelar a la API el cumpleaños o el nombre privado. Su implementación no forma parte de la corrección actual.

## Referencias

- Plan activo: `product/docs/plans/active/crush-age-birthday/README.md`
- Vault E2EE: `product/docs/plans/active/vault-e2ee/README.md`
