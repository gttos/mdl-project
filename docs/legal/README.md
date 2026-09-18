# Expediente legal y de privacidad de MDL

> **Destinatario:** abogado, DPO o especialista en privacidad que no conoce MDL.
> **Fecha de corte documental:** 14 de septiembre de 2026.
> **Modelo descrito:** funcionamiento de la beta de lanzamiento con Vault E2EE, verificación de email, Sign in with Apple y relaciones nativas por períodos.
> **Naturaleza:** descripción técnica y preguntas para revisión profesional; no contiene conclusiones jurídicas aprobadas.

## Propósito

Este expediente reúne la información que la asesoría necesita para preparar o revisar los textos y procedimientos de lanzamiento: qué hace MDL, qué personas y datos intervienen, dónde se guardan, quién puede leerlos, cómo se protegen, cuánto se pretende conservarlos y qué decisiones siguen correspondiendo a Legal.

El expediente describe el contrato funcional final de la beta. La evidencia de despliegue, configuración y operación se conserva separada: este documento no la sustituye ni convierte una decisión de Product en una conclusión jurídica.

## Resumen ejecutivo

MDL es un diario iOS privado sobre vínculos y experiencias afectivas. Gran parte de la información se refiere a terceras personas que normalmente no tienen cuenta y algunos campos permiten inferencias sobre vida afectiva o sexual.

El modelo de lanzamiento separa:

1. **Cuenta y datos funcionales legibles por API:** identidad de la cuenta, IDs, fechas, fases, etiquetas, motivos de cierre, historiales relacionales, valoraciones, asociaciones, rasgos y atributos necesarios para operar Timeline y estadísticas.
2. **Contenido privado E2EE:** nombre/contacto/redes/cumpleaños del crush, lugar/descripción de experiencias y nota de cierre. iOS lo cifra con una VMK por usuario antes de transferirlo; API y S3 reciben ciphertext y metadata, no VMK ni plaintext.
3. **Fotos locales:** permanecen en el dispositivo y no se sincronizan con MDL en la beta.
4. **Datos de seguridad y proveedores:** IP, user agent, eventos, trazas, dirección de email y metadata técnica según el flujo.
5. **Datos derivados:** `everMatched`, ratings, cronología y estadísticas personales; que sean derivados no los vuelve anónimos.

El modelo relacional de la beta usa períodos con fases, etiquetas opcionales durante Involucrado, motivos catalogados de cierre e historiales append-only de fases y etiquetas. API conserva esa estructura, fechas, asociaciones, hitos y el derivado monotónico `everMatched`; el texto libre permanece en Vault. Esta granularidad puede revelar o permitir inferir intimidad aunque no exista plaintext en backend.

## Cuestión crítica sobre “anonimización”

La baja sobrescribe identificadores directos, retira el Vault y conserva únicamente la estructura autorizada para estadísticas agregadas sin PII. Si permanecen filas enlazadas por UUID, secuencias o combinaciones raras, el resultado puede ser seudonimizado y seguir siendo dato personal. La asesoría debe calificar esa conservación y fijar finalidad, base, plazo, acceso y salvaguardas; los textos públicos no deben llamarla anónima sin esa evaluación.

## Orden de lectura

| Orden | Documento | Pregunta principal |
|---|---|---|
| 1 | [Contexto y entidades](01_MDL_Contexto_Producto_Entidades.md) | ¿Cómo funciona MDL y qué personas/entidades intervienen? |
| 2 | [Inventario de datos](02_MDL_Inventario_Maestro_Datos.md) | ¿Qué se procesa, dónde, para qué y quién puede leerlo? |
| 3 | [Vault y seguridad](03_MDL_Vault_Almacenamiento_Seguridad.md) | ¿Qué protege E2EE, qué metadata queda visible y qué debe demostrarse? |
| 4 | [Ciclo de vida y derechos](04_MDL_Ciclo_Vida_Retencion_Derechos.md) | ¿Cómo se actualiza, conserva, borra y atiende derechos? |
| 5 | [Guía de auditoría](05_MDL_Auditoria_Juridica.md) | ¿Qué decisiones y entregables se piden a la asesoría? |
| Anexo | [Respuesta a incidentes](anexos/respuesta-incidentes.md) | ¿Cómo clasificar, investigar y escalar una posible brecha? |

Como contexto funcional complementario puede consultarse el [blueprint de producto](../product/business-blueprint.md). Las reglas canónicas permanecen en [`PRODUCT.md`](../PRODUCT.md), [`VAULT.md`](../VAULT.md), [`TIMELINE.md`](../TIMELINE.md) y [`STATISTICS.md`](../STATISTICS.md).

## Entregables esperados de la asesoría

- criterio sobre responsable, encargados, destinatarios y alcance territorial;
- mapa de finalidades y bases por actividad, categoría de datos y tipo de persona;
- política de privacidad, capas informativas y cláusulas aplicables a terceras personas;
- revisión de términos en lo relativo a contenido aportado, Vault, recovery y baja;
- screening de EIPD y, si corresponde, EIPD completa antes del tratamiento;
- calendario de conservación y criterios de borrado/backups;
- procedimiento de acceso, rectificación, supresión, oposición, restricción y portabilidad;
- Registro de Actividades de Tratamiento;
- revisión de DPA, regiones, subencargados y transferencias;
- criterio sobre edad mínima y mecanismo de verificación proporcionado;
- procedimiento de incidentes y textos de notificación;
- límites de las afirmaciones públicas sobre E2EE, anonimización e IA.

## Cómo interpretar el expediente

- **Modelo de lanzamiento:** comportamiento funcional de la beta descrita.
- **Regla técnica observada:** configuración o comportamiento contrastado con código; Legal todavía puede exigir un cambio.
- **Evidencia pendiente:** configuración, prueba o contrato que operaciones debe aportar.
- **Pregunta jurídica:** decisión reservada a la asesoría.
- **Fuera de beta:** no forma parte del tratamiento activo aunque exista infraestructura parcial.

E2EE reduce la exposición del contenido privado, pero no elimina el tratamiento de datos estructurales, metadata, cuenta, dispositivo, proveedores ni información sobre terceras personas.

## Información empresarial todavía necesaria

- identidad, domicilio y datos de contacto del responsable;
- países y fecha prevista de lanzamiento;
- contacto operativo de privacidad y seguridad, representante y DPO si aplican;
- proveedores efectivamente contratados, entidades jurídicas, regiones, DPA, subencargados y transferencias;
- configuración real de RDS/S3, backups, versionado, logs, Sentry, SES y accesos;
- canal y sistema de tickets para derechos sin copiar contenido íntimo;
- textos de onboarding, recovery, archivo, borrado, baja e inactividad;
- calificación jurídica y salvaguardas de la estructura conservada para estadísticas después de la baja;
- evidencia de configuración y disclosure de verificación de email y Sign in with Apple; Google OAuth e IA no forman parte de la beta.

## Condición de uso externo

Este expediente puede entregarse al abogado como descripción y lista de decisiones. No debe convertirse directamente en política pública ni usarse como certificación de cumplimiento. La versión pública debe prepararse después de cerrar las preguntas jurídicas y adjuntar la evidencia operativa indicada.
