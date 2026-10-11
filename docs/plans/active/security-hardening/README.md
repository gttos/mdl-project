# Endurecimiento de seguridad de API

## Objetivo

Preparar la API de MDL para beta mediante pruebas de seguridad autorizadas, repetibles y trazables. La iniciativa convierte riesgos concretos de la API en reglas y pruebas; no simula ataques contra infraestructura real ni sustituye una revisión externa previa al lanzamiento.

## Alcance

- Autenticación, sesión, autorización por propietario, validación de entradas y salida pública de API.
- Flujos sensibles: registro, login, recuperación/verificación de email, Apple, Vault, URLs firmadas, Timeline y baja de cuenta.
- Límites de abuso, consumo de recursos, configuración expuesta y observabilidad segura.
- Pruebas locales in-process y, en una fase posterior aprobada de forma explícita, validación limitada contra staging descartable.

Fuera de alcance:

- Escanear, fuzzear o probar producción, AWS, Apple, S3 real, terceros o direcciones ajenas.
- Denegación de servicio, fuerza bruta, extracción de secretos, ingeniería social o uso de datos reales.
- Afirmar cumplimiento legal o reemplazar un pentest independiente.

## Reglas compartidas

- Todo acceso a un recurso identificado pertenece únicamente a su propietario. Un identificador de otra cuenta se rechaza sin revelar su existencia ni dejar mutaciones parciales. Esta regla concreta el aislamiento de [`ARCHITECTURE.md` §3](../../../ARCHITECTURE.md#3-identidad-y-aislamiento).
- Una ruta no autenticada existe sólo cuando su contrato la necesita; una sesión expirada, revocada o ajena no habilita funciones privadas.
- Los inputs aceptados se validan y autorizan en la frontera y se revalidan donde una transacción decide una invariante. Los atributos no admitidos no mutan estado.
- Respuestas y logs no exponen tokens, secretos, plaintext del Vault, PII innecesaria, SQL, stack traces ni detalles que permitan enumerar cuentas.
- Las rutas y flujos costosos o sensibles aplican límites proporcionados y comunican un rechazo temporal sin crear estado parcial.
- Las integraciones y URLs firmadas tratan toda respuesta externa como no confiable y mantienen los objetos privados.

## Escenarios canónicos

### GWT-SEC-01 — Aislamiento por objeto

**Dado** dos cuentas distintas y el identificador válido de un recurso de A
**Cuando** B intenta leerlo, modificarlo, eliminarlo o usarlo en una operación compuesta
**Entonces** API rechaza sin revelar datos de A ni persistir cambios.

### GWT-SEC-02 — Frontera de autenticación

**Dado** una ruta privada
**Cuando** llega sin token, con token vencido/revocado o de una cuenta distinta
**Entonces** no ejecuta su función ni devuelve datos privados.

### GWT-SEC-03 — Entrada y salida mínima

**Dado** un payload con campos no admitidos, IDs manipulados o datos privados
**Cuando** llega a una ruta API
**Entonces** sólo los campos permitidos pueden afectar el estado y la respuesta conserva el envelope público saneado.

### GWT-SEC-04 — Abuso de flujos sensibles

**Dado** un flujo de autenticación, OTP, baja, URL firmada o escritura sensible
**Cuando** supera su límite aplicable
**Entonces** devuelve un rechazo temporal utilizable y no deja una mutación parcial ni expone señales adicionales.

### GWT-SEC-05 — Fallo seguro y trazable

**Dado** un rechazo de autorización, validación, proveedor o excepción inesperada
**Cuando** API lo procesa
**Entonces** el cliente recibe un código estable saneado y la observabilidad conserva sólo metadata técnica permitida.

## Orden de fases

1. **PRODUCT/API-SEC-01 — Inventario y priorización:** mapear rutas, superficies y cobertura existente; fijar una matriz de amenazas dentro de estos README y seleccionar las primeras brechas demostrables. No cambia producción ni ejecuta tráfico externo.
2. **API-SEC-02 — Autenticación y autorización:** cerrar brechas priorizadas de GWT-SEC-01/02 con tests de Feature/Contract y cambios mínimos si hacen falta.
3. **API-SEC-03 — Abuso y configuración:** verificar y endurecer límites, tamaños, rutas e integraciones de GWT-SEC-04, sin activar una herramienta contra producción. Esta prioridad adelanta un override de migración confirmado que deja sin límite efectivo al grupo autenticado y a escrituras costosas de Vault.
4. **API-SEC-04 — Entrada, salida y observabilidad:** cubrir GWT-SEC-03/05, incluyendo errores, mass assignment y logs saneados.
5. **API-SEC-05A — Entorno local aislado:** preparar un stack Docker hermético y descartable para seguridad local: API, base de datos, correo capturado y almacenamiento compatible con S3, sin tocar AWS ni exponer un puerto fuera de `127.0.0.1`. Debe reproducir los flags y límites de seguridad relevantes de beta, nunca los valores relajados de desarrollo.
6. **API-SEC-05A-R — Retirada del detector legacy:** eliminar el middleware global inoperante `DetectSuspiciousBehavior`, su registro y su configuración exclusiva. Se conservan los limiters efectivos, el registro de `SecurityAlert` ante `429` y los consumidores operativos de esas alertas.
7. **API-SEC-05B — Validación local:** con autorización separada, ejecutar sólo comprobaciones locales contra ese stack y demostrar el comportamiento efectivo de los limiters y sus alertas. No hay tráfico contra una IP, dominio, proveedor ni cuenta externos.
8. **API-SEC-05C — Staging público aislado:** cuando se necesite probar desde iPhone, Apple o un scanner externo, desplegar el mismo stack en una máquina distinta de producción, con dominio/HTTPS, datos falsos, secreto, bucket y permisos propios.
9. **API-SEC-05D — Validación pre-beta externa:** con alcance escrito y autorización separada, ejecutar un DAST limitado contra el staging descartable, revisar dependencias y preparar el alcance de un pentest externo.

## Gates externos de la beta

El cierre del código de remediación no certifica la infraestructura. `API-SEC-05C` concentra desde ahora las comprobaciones que sólo pueden demostrarse en un entorno autorizado:

- rangos de proxy confiables, acceso directo al origin, HTTPS y cabeceras efectivas;
- recorrido Vault con el binario iOS vigente contra AWS S3 real, límites, checksum, expiración, cleanup, lifecycle, cuotas y recuperación;
- worker, scheduler, cola, cache con locks compartidos y configuración efectiva de Sentry/observabilidad;
- inventario de imágenes históricas y decisión separada de retirada o rotación si alguna incorporó material sensible;
- conteo técnico sin datos personales de marcas históricas de inactividad antes de activar el ciclo sobre un entorno existente;
- entrega real del aviso de inactividad, rebotes y Private Relay, además de aprobación humana/legal de su suficiencia;
- configuración y recorridos reales de Apple, correo y demás proveedores habilitados para la beta.

Estas comprobaciones son gates de despliegue u operación, no fases de código. Un scan estándar adicional queda a criterio del usuario; DAST o pentest conserva la autorización específica de `API-SEC-05D`.

## Criterios de aceptación

- [ ] El inventario clasifica cada ruta como pública, autenticada o sensible, y enlaza su control principal.
- [ ] Las superficies de mayor riesgo tienen pruebas negativas de aislamiento, autenticación o abuso según corresponda.
- [ ] Los gaps encontrados se priorizan por impacto y se corrigen mediante una fase explícita; no se ocultan como comportamiento aceptado.
- [ ] No se ejecutó tráfico de seguridad contra producción, proveedores ni datos reales.
- [ ] Antes de lanzamiento se completa una revisión independiente y autorizada sobre staging, además de la evidencia automatizada.

## Relación con iniciativas activas

- [Verificación de email](../../../rules/ACCOUNT.md#verificacion-email): OTP, cuenta provisional y gates de sesión.
- [`sign-in-with-apple`](../sign-in-with-apple/README.md): identidad federada, webhooks y revocación.
- [`vault-e2ee`](../vault-e2ee/README.md): ownership, blobs cifrados, bindings y URLs firmadas.

## Estado

API-SEC-01 (inventario), API-SEC-02 (frontera de sesión), API-SEC-03 (límites y configuración) y API-SEC-04 (respuesta pública genérica de recuperación) están implementadas y validadas localmente. `API-SEC-05A` está implementada: el stack Docker aislado quedó preparado y validado estáticamente, pero no se inició ni generó tráfico. `API-SEC-05A-R` está autorizada para retirar el detector legacy antes del primer arranque. `05B`, `05C` y `05D` requieren sus autorizaciones específicas posteriores.

Durante `05A` se confirmó un hallazgo de producción: `DetectSuspiciousBehavior` está registrado sobre el grupo `api`, pero una salida temprana incondicional desactiva su detección, anotación y bloqueo propios. No revierte los limiters explícitos restaurados en API-SEC-03; además, el manejo separado de un `429` generado por esos limiters conserva una vía para registrar `SecurityAlert`.

La decisión de producto es retirar ese detector legacy antes de iniciar `05B`, no reactivarlo ni conservar código muerto. `API-SEC-05A-R` elimina exclusivamente el middleware, su registro global, su bloque de configuración y las variables de ejemplo que sólo éste consume. Conserva `SecurityAlert`, el registro de alertas por limiters efectivos, los informes y sus demás consumidores. `05B` demuestra después el comportamiento que realmente queda. Hasta completar esa retirada y validación, la iniciativa de seguridad permanece abierta. No se hizo evaluación de seguridad ni tráfico de prueba externo.

## Planes locales

- [API](../../../../../api/docs/plans/active/security-hardening/README.md)
