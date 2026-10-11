# Correcciones detectadas durante la beta

> **Estado:** activo. Este plan es la bandeja única para defectos funcionales, visuales o de integración descubiertos al probar builds de beta. No modifica el estado ni las fases de otros planes activos mientras éstos se revisan o cierran.
>
> **Proyecto afectado actualmente:** iOS. API sólo se incorpora si una reproducción demuestra que el defecto cruza su contrato.

## Objetivo

Conservar en un único lugar los problemas nuevos observados durante las pruebas reales de beta, con evidencia, resultado esperado y validación de salida. Cada corrección se ejecuta en una fase propia y no se mezcla con iniciativas que ya están en validación o cierre.

Las reglas funcionales siguen viviendo en [`PRODUCT.md`](../../../PRODUCT.md) y `docs/rules/`. Este archivo registra trabajo pendiente; no redefine Cuenta, Apple, Vault, UX ni App Lock.

## Checklist vigente

| ID | Prioridad | Hallazgo | Estado | Salida exigida |
|---|---|---|---|---|
| `BETA-FIX-001` | Crítica | Activación de Face ID, App Lock y login biométrico presentan estados incoherentes y una contraseña ambigua. | Pendiente | Estado coherente en Settings/Login, cero cierre de sesión ante fallo y recorrido físico completo. |
| `BETA-FIX-002` | Alta | El vínculo de Apple con una cuenta MDL existente necesita el diseño de los formularios normales y distinguir la contraseña de acceso de la del Vault. | Pendiente | Formulario común, copy inequívoco, errores recuperables y validación física con email coincidente y Private Relay. |

## `BETA-FIX-001` — Face ID, App Lock y login biométrico

### Evidencia observada

- Settings pidió una contraseña al intentar activar Face ID sin dejar claro qué credencial correspondía.
- Introducir la contraseña del Vault terminó la sesión y devolvió a la entrada pública.
- Después del login apareció Face ID, pero Settings continuó mostrando la opción desactivada.
- Tras logout, el login no ofreció Face ID.

La inspección del `develop` local confirma una causa parcial:

- `AuthViewModel.isBiometricsEnabled` se inicializa desde `UserDefaults`;
- `verifyPasswordAndEnableBiometrics` guarda email/password, pero no activa ni persiste ese estado;
- `disableBiometrics` elimina las credenciales, pero tampoco desactiva ni persiste el estado;
- Settings y Login dependen de ese booleano, por lo que pueden mostrar resultados incompatibles;
- el Face ID que aparece después de autenticar pertenece al App Lock del Vault: `AppLockOverlayWindowCoordinator` llama automáticamente a `VaultUIManager.unlockWithBiometrics()` cuando el estado local lo exige y no demuestra que el login biométrico esté habilitado.

El cierre de sesión todavía requiere reproducción dirigida: el flujo de activación no debe descartar una sesión válida ante contraseña incorrecta, cancelación o error local.

### Resultado esperado

- App Lock y login biométrico son recorridos distintos. App Lock cubre y desbloquea contenido local; no inicia sesión ni activa por sí solo el login con Face ID.
- Para activar el login biométrico se solicita explícitamente la **contraseña de acceso a MDL**, nunca la contraseña del Vault.
- La contraseña del Vault se usa al configurar su protección y para recuperar/reconstruir el Vault; no participa en login, vínculo Apple ni activación del login biométrico.
- Una contraseña incorrecta, cancelación o fallo de Keychain/biometría conserva sesión, token, namespace, Vault, navegación y estado anterior.
- El éxito guarda las credenciales protegidas y persiste el estado activado como una única transición. Settings lo muestra activo inmediatamente y tras relanzar.
- Después de logout, Login ofrece Face ID/Touch ID y, al usarlo, ejecuta el bootstrap normal del mismo usuario y Vault.
- Desactivar elimina credenciales y estado. Un cambio de cuenta no reutiliza silenciosamente material de otra cuenta.
- Una cuenta Apple-only mantiene App Lock, pero no ofrece login biométrico basado en contraseña hasta añadir una contraseña MDL.

### Implementación y pruebas

- Sustituir sleeps/polling de Settings por una operación async con resultado explícito.
- Centralizar credenciales Keychain y estado persistido para que activar/desactivar sea coherente.
- Verificar aislamiento por cuenta, logout, cambio de cuenta y relanzamiento.
- Mantener separados `unlockWithBiometrics()` y `loginWithBiometrics()`.
- Cubrir activación correcta/incorrecta/cancelada, error de Keychain, desactivación, Apple-only, cambio de cuenta, persistencia, logout→Face ID y cero teardown ante fallo.
- Recorrer en dispositivo físico: sesión abierta, background/foreground, contraseña correcta e incorrecta, cancelación, logout, relanzamiento y biometría no disponible.

## `BETA-FIX-002` — formulario de vínculo con Apple

### Evidencia observada

Una cuenta MDL ya existía con el mismo email. Al usar Continuar con Apple apareció una pantalla de vínculo sin el acabado de los formularios normales y pidió email/password sin diferenciar con suficiente claridad la contraseña de acceso de la contraseña del Vault.

El `develop` local contiene una adopción parcial de `FlowBaseView`, foco y validación común que todavía no está aceptada ni distribuida. Debe revisarse y reutilizarse; su mera presencia no cierra el hallazgo.

### Resultado esperado

- La pantalla usa el mismo contenedor, campos, bordes, foco, validación, CTA, loading y error que los formularios normales con tamaño de texto normal.
- Los campos se identifican como **email de tu cuenta MDL** y **contraseña de acceso a MDL**.
- El copy aclara que no se solicita la contraseña del Vault.
- La contraseña incorrecta conserva email, continuación Apple y posibilidad de corregir; no crea otra cuenta, vínculo, sesión ni Vault.
- Cancelar no presenta un error técnico ni reinicia innecesariamente la autorización.
- Se prueban email Apple coincidente y Private Relay/email distinto mediante el recorrido explícito «Ya tengo una cuenta MDL».

### Implementación y pruebas

- Contrastar el `AppleLinkView` actual con el patrón visual de Login/Registro.
- Cubrir formulario ES/EN, campos requeridos, Return y CTA convergentes, double submit, error recuperable y cero sesión/vínculo ante contraseña incorrecta.
- Comprobar que copy y navegación no remiten al recovery del Vault.
- Validar físicamente contraseña correcta/incorrecta, cancelación, teclado, email coincidente y Private Relay. Lectores de pantalla y texto ampliado quedan fuera por `RP-UX-006`.

## Orden de ejecución

1. `BETA-FIX-001`: corrige una frontera de sesión y seguridad observable.
2. `BETA-FIX-002`: corrige diseño y semántica del vínculo Apple.
3. Smoke físico conjunto: App Lock, activación/desactivación biométrica, logout/login y vínculo Apple.

Cada punto se implementa en una sesión independiente. Al aceptar una corrección se sustituye su estado en esta tabla por la evidencia vigente; no se crea un reporte adicional. Cuando no queden fixes abiertos, los hechos estables se integran en la documentación canónica aplicable y este plan se elimina conforme a `AGENTS.md`.

## Decisiones abiertas

No hay una decisión funcional pendiente para estos dos hallazgos. La separación entre contraseña de cuenta, contraseña del Vault y App Lock ya está definida; falta corregir y validar la implementación observable.
