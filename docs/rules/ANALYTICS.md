# Reglas de producto — Analítica de uso en iOS

**Estado de revisión: FINALIZADO.** Producto ha aprobado el modelo funcional de analítica para iOS. Este estado no autoriza el proveedor ni el rollout productivo: `DEC-15`, `DEC-09` y la revisión humana/legal permanecen abiertas.

[Índice de reglas](../PRODUCT.md) · Medición agregada de pantallas y recorridos para mejorar MDL sin publicidad, perfiles individuales ni captura de contenido.

<a id="alcance"></a>
## Finalidad y alcance

- **RP-ANA-001.** La beta puede medir en iOS el uso agregado de pantallas y recorridos para decidir qué mejorar en login, registro, creación y edición del diario, fotos, Timeline y Estadísticas. Esta es una excepción de plataforma explícita a `RP-BASE-008`: Android, API y LLM no incorporan analítica de producto por esta iniciativa.
- **RP-ANA-002.** La analítica sirve exclusivamente para producto. Quedan fuera publicidad, atribución publicitaria, IDFA, App Tracking Transparency, enlace entre aplicaciones o empresas, compra o venta de audiencias, perfiles individuales, session replay, grabaciones, screenshots, heatmaps, autocaptura de vistas o gestos y captura indiscriminada de cada toque.
- **RP-ANA-003.** La medición registra acciones funcionales intencionales y resultados reales; no intenta reconstruir el contenido del diario ni observar el comportamiento de una persona concreta. La indisponibilidad o el fallo de la analítica nunca impide abrir una pantalla, guardar, sincronizar, iniciar sesión ni completar otro comportamiento de MDL.

<a id="activacion"></a>
## Activación, información y control

- **RP-ANA-004.** Después de aprobarse `DEC-15`, `DEC-09`, el disclosure público y la configuración de privacidad del proveedor, la analítica se habilita por defecto en iOS. Esto no es un opt-in: la persona no tiene que activarla. Antes de la primera emisión, MDL comunica de forma clara la finalidad, las categorías medidas y cómo desactivarla.
- **RP-ANA-005.** Ajustes ofrece siempre un control comprensible para desactivar o volver a activar la analítica. Desactivarla detiene cualquier emisión futura y elimina del dispositivo eventos pendientes y correlación efímera de analítica. No cambia la cuenta, el Vault, el diario ni las funciones disponibles.
- **RP-ANA-006.** Mientras `DEC-15`, `DEC-09` o la revisión humana/legal estén pendientes, iOS puede implementar y probar el contrato con un receptor en memoria o sin salida, pero ninguna build de producción envía eventos a un proveedor real. La ausencia de configuración falla cerrada y mantiene desactivada la emisión externa.

<a id="datos"></a>
## Datos permitidos y prohibidos

- **RP-ANA-007.** Cada evento pertenece a un catálogo cerrado y sólo admite dimensiones enumeradas necesarias para la métrica: pantalla, flujo, paso, punto de entrada, método de acceso, acción, resultado, categoría de error saneada, estado de contenido, sección, tipo de elemento y tramo de cantidad. También puede incluir versión/build de la app, versión mayor de iOS e idioma de interfaz.
- **RP-ANA-008.** La analítica nunca incluye identificadores de cuenta, dispositivo publicitario, Crush, experiencia, relación, Vault, foto u operación; email, fecha de nacimiento, género, país ni otros datos de perfil; nombres, fechas o texto introducido; satisfacción, valoraciones, situación relacional, resultado de match o valores estadísticos; tokens, rutas/URLs con parámetros, payloads, ciphertext, fotos ni contenido del Vault.
- **RP-ANA-009.** Mensajes libres de error, `localizedDescription`, excepciones y respuestas de red no son propiedades analíticas. Los fallos se traducen antes de medirlos a una categoría cerrada que no permita recuperar el contenido ni la identidad afectada.
- **RP-ANA-010.** Para calcular un embudo dentro de una ejecución puede utilizarse una correlación aleatoria y efímera de sesión únicamente si `DEC-15` la aprueba. No se vincula con la cuenta, Keychain, Vault ni identificadores del diario; no sobrevive al límite de sesión aprobado y se destruye al desactivar la analítica. No se crea un identificador analítico persistente ni un perfil entre sesiones.

<a id="semantica"></a>
## Semántica de los eventos

- **RP-ANA-011.** Una vista de pantalla se registra cuando la pantalla queda realmente visible. Recomputaciones de SwiftUI, reaparecer un componente hijo o reconstruir el árbol sin una nueva visita no generan otra vista.
- **RP-ANA-012.** Los recorridos distinguen inicio, paso visible, envío, guardado durable local, sincronización remota, finalización, cancelación y fallo. Un evento de finalización sólo se emite tras alcanzar la frontera funcional que su nombre declara; pulsar un botón o iniciar una petición no equivale a completar el recorrido.
- **RP-ANA-013.** El registro se considera completado sólo cuando existen la cuenta confirmada, la autenticación requerida y la sesión MDL publicada; en email incluye la verificación OTP. Solicitar el alta, recibir una capacidad provisional o avanzar de pantalla no es `registration_completed`.
- **RP-ANA-014.** Creación y edición distinguen el guardado durable en el dispositivo de la confirmación de sincronización. Timeline y Estadísticas miden que una superficie o sección fue vista y las acciones funcionales previstas, nunca el contenido, los valores presentados ni el identificador del elemento abierto.
- **RP-ANA-015.** La telemetría técnica de red, seguridad, rendimiento y diagnóstico —incluido rate limiting— no es analítica de producto. Usa el canal técnico permitido por `RP-SEG-006` y `RP-SEG-007`, con su propia retención y sin reutilizar nombres de eventos de producto.

<a id="calidad"></a>
## Calidad y uso de las métricas

- **RP-ANA-016.** Los informes muestran agregados de sesiones, vistas, inicios, resultados y conversión. MDL no ofrece búsquedas, fichas ni recorridos de una persona concreta y no utiliza estos datos para decisiones automatizadas sobre cuentas.
- **RP-ANA-017.** Todo evento nuevo exige definición de finalidad, frontera de emisión, propiedades permitidas, métrica derivada y prueba que demuestre emisión exacta y ausencia de datos prohibidos. Renombrar o cambiar la semántica exige versionar el contrato o coordinar la migración del informe; no se reutiliza un nombre para otro significado.
- **RP-ANA-018.** Los dashboards diferencian ausencia de uso de ausencia de instrumentación. Ninguna decisión de producto se apoya en una métrica hasta verificar cobertura, volumen esperado, duplicados, opt-out y funcionamiento de la versión de app analizada.

<a id="decisiones-pendientes"></a>
## Decisiones pendientes

| ID | Decisión pendiente | Lo ya fijado |
|---|---|---|
| <a id="dec-15"></a>DEC-15 | Aprobar proveedor, región y transferencias, contrato/DPA y subencargados, configuración de privacidad, disclosure y ficha App Store Privacy; concretar el límite de la correlación efímera, retención cruda/agregada, borrado y evidencia del opt-out junto con `DEC-09`. | El alcance es sólo analítica de producto iOS; estará habilitada por defecto tras la aprobación, con información clara y opt-out; no habrá publicidad, tracking entre apps, perfiles individuales, replay, autocaptura, IDs de negocio ni contenido privado. Hasta resolverlo, la salida productiva a un proveedor permanece desactivada. |
