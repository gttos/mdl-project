# Expediente legal y de privacidad de MDL

> **Destinatario:** abogado, DPO o especialista en privacidad que no conoce previamente MDL.
> **Modelo descrito:** estado actual del producto en desarrollo, previo a su lanzamiento público en Europa y previo a subir la API vigente a producción.
> **Fecha de referencia:** 5 de agosto de 2026.
> **Naturaleza:** descripción factual del sistema tal como funciona hoy, verificada contra el código de `api/` e `ios/`. No contiene conclusiones jurídicas aprobadas ni describe un estado futuro o planificado.

## Propósito del expediente

Este expediente permite entender qué es MDL, qué datos utiliza hoy, por qué los necesita, dónde se guardan, quién puede leerlos, cómo se protegen y qué ocurre durante su ciclo de vida. Su objetivo es reducir el tiempo de descubrimiento técnico de la asesoría y darle una base ordenada para:

- evaluar riesgos para usuarios y terceras personas antes de que la API actual reciba tráfico de producción;
- determinar bases jurídicas, deberes de información y necesidad de una EIPD;
- redactar la política de privacidad, términos y textos de consentimiento o aceptación;
- revisar proveedores, transferencias internacionales, conservación y ejercicio de derechos;
- priorizar correcciones técnicas antes del lanzamiento.

## Resumen ejecutivo

MDL es una aplicación iOS de registro personal de vínculos y experiencias afectivas. El usuario crea perfiles privados de personas que le interesan, registra experiencias y mantiene el historial de sus relaciones. La aplicación construye un timeline y métricas personales a partir de esos registros.

La característica jurídica principal es que muchos datos se refieren a personas que no tienen cuenta ni relación directa con MDL. Además, algunas valoraciones y textos pueden revelar información íntima o permitir inferencias sensibles.

**Hoy conviven dos mecanismos de protección para el mismo contenido privado, no uno solo:**

1. **Columnas funcionales cifradas con una única clave de aplicación.** El nombre, contacto y redes del crush, el lugar y la descripción de una experiencia, y la nota de cierre de una relación existen como columnas en la base relacional. La mayoría de esas columnas están cifradas con `Crypt::encryptString` de Laravel usando la clave de aplicación (`APP_KEY`); el backend descifra ese contenido de forma transparente cada vez que lo lee. El nombre del crush (`Crush.name`) **no** está cifrado y se guarda en texto plano. Una sola clave protege el contenido de todas las cuentas.
2. **Vault E2EE, en construcción y activo solo parcialmente.** La app iOS genera una clave por usuario (VMK), cifra el mismo contenido en el dispositivo y sube el ciphertext a un almacenamiento de objetos separado. El backend guarda metadata y el ciphertext, pero no la clave ni el contenido descifrado, únicamente para las entradas que ya pasaron por este flujo.

Ninguno de los dos mecanismos sustituye hoy al otro: las columnas cifradas con la clave de aplicación siguen existiendo, se siguen escribiendo y los endpoints de la API todavía pueden devolver su contenido descifrado. El proceso que debía limpiarlas tras la migración a Vault (`CleanupLegacyDataJob`) solo registra que se ejecutó y no modifica ninguna columna.

El producto separa la información en tres capas:

1. **Datos funcionales legibles por el backend:** cuenta, identificadores, fechas, estados, clasificaciones, valoraciones y eventos técnicos necesarios para operar el servicio.
2. **Contenido privado (columnas cifradas con clave única + Vault parcial):** nombre y contacto del crush, lugar y descripción de una experiencia y nota de cierre de una relación. Hoy este contenido es recuperable por el backend a través de las columnas funcionales, y adicionalmente existe una copia cifrada con VMK en el Vault para las entradas ya migradas.
3. **Datos exclusivamente locales:** fotografías del crush, notas seguras sin sincronización remota y el índice de búsqueda local. Las fotografías no se sincronizan con MDL; el índice de búsqueda local guarda una copia en texto plano del mismo contenido privado dentro del dispositivo.

## Orden de lectura recomendado

| Orden | Documento | Responde principalmente |
|---|---|---|
| 1 | [Contexto del producto y entidades](01_MDL_Contexto_Producto_Entidades.md) | ¿Qué es MDL y cómo lo usa una persona hoy? |
| 2 | [Inventario maestro de datos](02_MDL_Inventario_Maestro_Datos.md) | ¿Qué guarda cada entidad, dónde y para qué, hoy? |
| 3 | [Vault, almacenamiento y seguridad](03_MDL_Vault_Almacenamiento_Seguridad.md) | ¿Qué puede leer cada sistema hoy y qué controles existen realmente? |
| 4 | [Ciclo de vida, conservación y derechos](04_MDL_Ciclo_Vida_Retencion_Derechos.md) | ¿Cuánto se conserva, cómo se borra hoy y cómo se atienden derechos? |
| 5 | [Guía para la auditoría jurídica](05_MDL_Auditoria_Juridica.md) | ¿Qué debe revisar y responder la asesoría antes de producción? |
| Anexo | [Respuesta a incidentes](anexos/respuesta-incidentes.md) | ¿Cómo se investiga y escala una posible brecha hoy? |

## Cómo interpretar las afirmaciones

| Marca | Significado |
|---|---|
| **Estado verificado** | Comportamiento confirmado contra el código de `api/` o `ios/` en la fecha de referencia. |
| **Evidencia pendiente** | Afirmación tomada de documentación de producto que no fue releída línea por línea contra el código en esta revisión; debe confirmarse antes de firmar. |
| **Pregunta para asesoría** | Cuestión abierta que requiere criterio profesional; el expediente no adopta la respuesta. |

Esta documentación describe **el estado actual del sistema**, incluyendo comportamientos incompletos, contradictorios o pendientes de corrección. No describe un estado futuro deseado. Cuando el Vault E2EE quede completo y las columnas legacy se retiren, este expediente debe revisarse de nuevo; hasta entonces, la asesoría debe evaluar el sistema tal como está descrito acá.

## Alcance canónico

`product/docs/legal/` es la única fuente documental preparada para revisión jurídica. Las especificaciones técnicas de API e iOS sirven como evidencia interna, pero no sustituyen este expediente ni deben entregarse como explicación inicial.

El inventario privado está limitado a tres entidades: **crush**, **experiencia** y **relación**. Cualquier nuevo tipo de contenido, proveedor o tratamiento requerirá actualizar primero este expediente.

## Información que debe adjuntarse antes de enviarlo al abogado

- identidad y domicilio de la sociedad responsable;
- países de lanzamiento y edad mínima propuesta;
- lista real de proveedores de producción, regiones, contratos y DPA (incluyendo Google, Amazon Web Services, Sentry y Apple);
- configuraciones de conservación, backups, subencargados y transferencias;
- evidencia técnica de qué campos concretos deja de ver el backend una vez que el Vault se complete, y con qué fecha se propone retirar las columnas cifradas con clave única;
- versiones preliminares de los flujos de alta, información, aceptación, baja y ejercicio de derechos;
- contacto operativo de privacidad y seguridad.
