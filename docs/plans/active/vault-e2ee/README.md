# Private Journal Vault (E2EE)

> **Estado:** activo; cierre técnico en curso
> **Proyectos:** iOS y API
> **Última actualización:** 2026-08-05

## Objetivo

El texto privado del usuario se cifra en su dispositivo y solo puede leerse después de desbloquear o recuperar el Vault. La API conserva estructura funcional y blobs opacos, pero no recibe ni interpreta el contenido privado.

## Alcance

- Datos privados del perfil de un crush.
- Lugar y descripción de cada experiencia.
- Nota de cierre de cada relación.
- Sincronización, versionado, migración y recuperación E2EE.
- Búsqueda local por nombre y borrado completo reintentable.
- Fotos sanitizadas y almacenadas únicamente en el dispositivo.

Fuera de alcance:

- Nuevas clases de contenido privado distintas de crush, experiencia y relación.
- **Backup o sincronización remota de fotos y videos.** El Vault cubre exclusivamente el contenido sensible de texto (nombre, contacto, redes, cumpleaños, descripción, notas de cierre). Las fotos y videos de un crush quedan fuera de su alcance: viven solo en el dispositivo, sin copia remota, y se pierden si se pierde o se resetea el dispositivo. Es una exclusión deliberada para poder lanzar el producto ahora, no un olvido — está registrada en `product/docs/plans/completed/photo-backup-hold/README.md` y en la fase `iOS-10`, eliminada del MVP. Retomarlo más adelante es viable como un backfill silencioso (sin acción del usuario) reutilizando el mismo motor de migración que ya usan crush/experiencia/relación; no requiere una decisión de arquitectura nueva cuando se retome, y no forma parte de esta fase.
- Interpretación del ciphertext por la API.

## Comportamiento esperado

### Escritura y lectura

- Cada crush, experiencia y relación tiene una entrada privada independiente identificada por su tipo e ID funcional.
- Crear o editar una entidad solo modifica su propia entrada.
- Los endpoints funcionales transportan IDs, fechas, estados y clasificaciones; no transportan nombre privado, lugar, descripción ni nota de cierre.
- El timeline obtiene su estructura desde API y superpone el contenido descifrado por tipo e ID.
- Una entidad con `is_anonymized = true` permanece excluida de Home y de cualquier superficie que la regla de anonimización ya oculte; Vault no revierte esa decisión de privacidad.
- El cleanup de texto legacy no debe activar `is_anonymized`: migración E2EE y anonimización funcional son operaciones distintas.
- Los flags de presentación local, como `isFromVault`, pertenecen exclusivamente a iOS, no forman parte del contrato API y su ausencia nunca invalida la decodificación del payload.
- Una fecha de nacimiento desconocida permanece ausente. La migración no inventa `1 de enero`, `31 de diciembre` ni otra fecha a partir de la edad. El formulario actual solo recoge día y mes: iOS los guarda como cumpleaños parcial y calcula el zodiaco, pero no deriva ni presenta un año que el usuario no introdujo. La semántica de edad confirmada y su ancla vive en `product/docs/plans/active/crush-age-birthday/README.md`.
- Editar contenido privado conserva una sola entry lógica por entidad. La UI no anuncia éxito ni invalida otras pantallas hasta que la actualización local cifrada haya terminado; una escritura no puede crear una segunda entry para la misma entidad.
- En altas y ediciones, `Añadido` o `Guardado` significa operación completa: escritura estructural aceptada por API, contenido cifrado persistido localmente y todas las entries Vault requeridas registradas, subidas y verificadas remotamente. Mientras falte cualquiera de esos pasos, la UI permanece en `Enviando…` y no publica éxito, refresca ni cierra el formulario.
- Si un paso posterior a la escritura estructural falla, la UI informa que la operación no se completó y ofrece `Reintentar`. El reintento reanuda el paso pendiente con los mismos IDs funcionales y de blob; no repite ciegamente una creación ya aceptada ni crea entidades o entries duplicadas.

### Sincronización y versiones

- Cada versión cifrada usa un ID remoto aleatorio distinto del ID funcional.
- Una versión anterior permanece recuperable hasta confirmar que la nueva versión existe remotamente.
- Los reintentos continúan sobre el mismo blob registrado.
- El tipo, ID funcional, asociación, revisión y versión del formato permanecen dentro del ciphertext.

### Migración y recuperación

- La migración no se considera completa por la mera existencia de blobs; debe comparar el inventario exacto de crushes, experiencias y relaciones.
- Los datos generados por una ejecución defectuosa no se reparan fila a fila: se crea y verifica una nueva revisión completa desde legacy y solo entonces se activa como conjunto vigente.
- Un fallo parcial queda pendiente y puede reanudarse sin borrar entries válidas.
- La validación y recuperación descargan y descifran sin borrar blobs.
- Contraseña de recuperación, frase de 24 palabras y QR deben reconstruir la misma VMK y el mismo conjunto de entidades en un dispositivo limpio.

### Búsqueda y privacidad local

- La búsqueda textual de la lista se realiza por nombre descifrado en memoria.
- El término no se envía a API ni se conserva en un índice persistente.
- Los resultados descifrados se descartan al bloquear o enviar la app a background.
- No se registra contenido privado en logs.
- Cada almacenamiento local y cada referencia de Keychain quedan vinculados al identificador estable del usuario autenticado. Cerrar sesión bloquea el Vault y elimina claves y contenido descifrado de memoria; iniciar otra cuenta nunca reutiliza el SQLite, la VMK ni los artefactos locales de la cuenta anterior.

### Borrado y fotos

- Un borrado conserva las referencias necesarias hasta que API y almacenamiento remoto confirmen la operación.
- Los fallos quedan reintentables y no se presentan como éxito.
- Al finalizar el borrado se eliminan las entries locales y la carpeta local de fotos del crush.
- Las fotos no generan tráfico de Vault/API y quedan excluidas del backup general del sistema.

## Interacción entre proyectos

| Proyecto | Responsabilidad funcional |
|---|---|
| iOS | Generar IDs funcionales, cifrar/descifrar, gestionar versiones, inventario, búsqueda local, recovery y fotos locales |
| API | Persistir estructura funcional, encapsulaciones y metadata opaca; entregar URLs; verificar objetos; autorizar por propietario |

La API no conoce el tipo o ID funcional contenido en un blob. iOS no considera autorizada ninguna limpieza legacy hasta demostrar recuperación completa.

## Orden general

1. Estabilizar las lecturas Vault y Home únicamente en iOS mediante iOS-09.1a, sin cambiar la semántica de anonimización de API.
2. Validar las escrituras iOS ya implementadas contra los contratos API completados.
3. Incorporar el envelope tipado y versionado.
4. Aislar el almacenamiento local y el material criptográfico por usuario autenticado.
5. Corregir migración, validación remota y recovery no destructivo.
6. Cerrar privacidad local, búsqueda, borrado, fotos y recuperación mnemónica.
7. Ejecutar validación transversal con datos legacy intactos.
8. Limpiar datos legacy únicamente con backup, evidencia y aprobación humana.
9. Repetir la validación con valores legacy vacíos antes de retirar outputs o columnas.

## Criterios de aceptación

- [ ] Varias experiencias y relaciones del mismo crush sobreviven a sync y recovery sin colisiones.
- [ ] Crear o editar no envía texto privado por endpoints funcionales.
- [ ] Un dispositivo limpio reconstruye perfil y timeline completos por tipo e ID.
- [ ] Recovery y validación realizan cero borrados remotos.
- [ ] No hay texto privado en logs, índices persistentes o requests funcionales.
- [ ] Dos cuentas usadas en una misma instalación nunca comparten SQLite, VMK, Keychain, media local ni estado descifrado.
- [ ] La búsqueda por nombre funciona sin enviar el término a API.
- [ ] Home excluye entidades realmente anonimizadas y, para las no anonimizadas, muestra valores privados únicamente desde Vault.
- [ ] Editar día/mes actualiza el cumpleaños parcial y el zodiaco sin inventar un año, conserva una única entry del crush y refresca perfil y Home solo después de persistir el Vault.
- [ ] Altas y ediciones no muestran éxito hasta verificar remotamente todas sus entries Vault; un fallo permite reintentar desde el paso pendiente sin duplicar la escritura estructural ni las entries.
- [ ] La migración conserva fechas legacy reales y deja como desconocidas las ausentes; no aparecen fechas sintéticas `31/12` o `01/01`.
- [ ] Los tres métodos de recovery recuperan el mismo Vault.
- [ ] Un borrado fallido se puede reintentar sin perder referencias.
- [ ] Las fotos siguen locales, excluidas de backup y se eliminan con el crush.
- [ ] La limpieza legacy solo ocurre después de la aprobación explícita.
- [ ] Abandonar el onboarding del Vault antes de confirmar haber guardado la frase de recuperación es reanudable en cualquier sesión posterior regenerando contraseña, VMK y frase desde cero, sin duplicar el alta remota del vault. Abandonarlo después de confirmar la frase pero antes de terminar la migración retoma directamente la migración, pidiendo Face ID como corresponde a un vault ya confirmado, sin volver a mostrar contraseña ni frase.
- [ ] La recuperación por frase de 24 palabras reconstruye el mismo Vault que la contraseña, contra un vault creado en cualquier momento.

## Estado resumido

| Proyecto | Estado | Próximo resultado |
|---|---|---|
| API | API-05.1 y API-05.3a completadas. API-10 completada: tabla `vault_mnemonic_encapsulations` (una fila por vault) y `GET`/`PUT /vault/encapsulation/mnemonic`, validado con 32 tests de Vault (154 assertions); `PATCH /vault/encapsulation` confirmado por test dedicado como seguro para reemplazos repetidos | API-05.3b después de cerrar el tráfico iOS |
| iOS | iOS-09.1 a iOS-09.3b completadas. iOS-11.1 implementada en código: `VaultState` distingue onboarding sin confirmar (reinicia con `PATCH /vault/encapsulation` + `PUT` mnemónico) de migración pendiente confirmada (retoma con Face ID); `startRecoveryWithMnemonic` ya lee la encapsulación mnemónica real en vez de la de contraseña. Sin build ni tests corridos todavía | Validar iOS-11.1 con build y prueba manual end-to-end (reinicio de onboarding, retomar migración, recovery con 24 palabras) antes de marcarla completada |
| Transversal | API-10 cerrada con evidencia real | Falta validar iOS-11.1 en dispositivo/simulador para dar por robusto el alta de cuentas nuevas y la recuperación por mnemónico |

## Planes locales

- API: `api/docs/plans/active/vault-e2ee/README.md`
- iOS: `ios/docs/plans/active/vault-e2ee/README.md`

## Referencias estables

- Arquitectura: `product/docs/architecture/system-overview.md`
- Comportamiento actual: `product/docs/product/media-and-vault.md`
- Decisión E2EE: `product/docs/decisions/vault-e2ee-architecture.md`
- Migración: `product/docs/migrations/vault-data-migration.md`
- Edad y cumpleaños: `product/docs/plans/active/crush-age-birthday/README.md`
- Legal: `product/docs/legal/README.md`
