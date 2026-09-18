# Entorno de integración local — plan maestro

**Estado:** planificado; no se ha iniciado código de esta iniciativa. Se implementará después de cerrar `relationship-redesign`, sin trasladar sus gates de beta a este plan. La prueba aislada que ya está en curso —incluido el primer Crush pendiente de sincronizar— permanece en [Relationship](../relationship-redesign/README.md) hasta concluirla.

## Objetivo y límites

La persona desarrolladora debe poder arrancar en su Mac una API aislada y abrir desde Xcode una app iOS local reconocible, probar flujos reales con datos ficticios y volver a hacerlo al día siguiente sin desplegar ni crear cuentas en producción. El estado local cotidiano debe sobrevivir a una parada normal y no mezclarse con el stack efímero de seguridad, el stack de desarrollo existente ni con sesiones/Vault de la app de producción.

No se crea staging público ahora. Este entorno no reemplaza tests dirigidos, comprobaciones en dispositivo físico, revisión Legal, ni el gate de despliegue de octubre. No convierte producción en un destino de pruebas sintéticas.

## Reglas y escenarios compartidos

| Regla | Dado/Cuando/Entonces |
|---|---|
| Local seguro | Dada la variante iOS local en simulador, cuando se ejecuta desde Xcode o se reabre desde su icono, entonces todas sus llamadas funcionales y de Vault siguen en loopback; la app normal y Release mantienen producción. La variante local tiene almacenamiento/sesión separados y se distingue visualmente. |
| Backend cotidiano | Dado el backend local detenido, cuando se inicia o reconstruye con el comando cotidiano, entonces sólo cambia su proyecto aislado, conserva datos en una parada ordinaria y falla cerrado ante conflicto de puertos; no ejecuta un reset implícito ni toca AWS, desarrollo o producción. |
| Escenario vacío | Dada una cuenta ficticia verificada sin Crushes, cuando abre Home, Timeline y Dashboard, entonces se muestran estados vacíos auténticos, no números o filas fabricados. |
| Escenario poblado | Dada otra cuenta ficticia con alrededor de cuatro Crushes, períodos vivos/cerrados, Experiences y un cambio compuesto, cuando abre, edita y reabre la app, entonces el contenido privado es recuperable desde Vault E2EE, las operaciones completan su sincronización y las lecturas son coherentes. Sólo insertar filas funcionales en MySQL no constituye un escenario válido. |
| Mensajes diarios | Dado el backend local, cuando Home solicita los *daily hacks* en español o inglés, entonces recibe al menos un tip sintético activo por categoría con ambos textos y la app muestra el carrusel; una lista vacía sólo es válida si se prueba expresamente el estado sin contenido. |
| Aislamiento | Dadas las dos cuentas, ninguna puede ver o editar entidades de la otra; credenciales, tokens y claves de Vault no se versionan ni se copian a informes. |

## Orden de fases

1. **LOCAL-01 — operación y cliente diario:** API prepara un stack persistente propio con arranque/estado/rebuild y correo local visible; iOS prepara una variante local seleccionable desde Xcode, fijada por build y separada de la app normal. Las fases locales pueden avanzar en paralelo sólo después de terminar el recorrido pendiente de Relationship, con un agente por proyecto y sin archivos compartidos.
2. **LOCAL-02 — escenarios repetibles:** API prepara catálogos y material legal sintético únicamente local; API/iOS acuerdan cómo crear o restaurar dos cuentas ficticias y contenido privado con Vault cifrado real. La restauración destructiva es una acción separada, explícita y acotada al entorno identificado.
3. **LOCAL-03 — rutina corta:** dejar instrucciones de uso cotidiano y un recorrido de verificación vacío/poblado, edición, Timeline y Dashboard; separar evidencia automatizada, manual local y física. Cada fase ejecuta sólo tests de los símbolos/rutas modificados, más la regresión proporcional a su riesgo.

## Dependencias y salida

- El modo actual `--integration-local-api` sólo aplica al lanzamiento Debug/simulador que recibe el argumento; reabrir la misma app desde el icono sin argumento vuelve al destino normal. Por eso es temporal y no satisface la regla de «Local seguro».
- El stack `mdl-security-local` y su bridge Vault son una referencia técnica validada, pero su ciclo efímero no debe ser la base persistente de demostraciones.
- El seed poblado espera una escritura real con upload/verificación Vault completos; no se diseñan fixtures que disimulen un `Syncing…` permanente.
- Salida de la iniciativa: rutina diaria documentada y reproducible sin producción, cuentas/escenarios locales editables, aislamiento demostrado y transferencia de la operación estable a los documentos canónicos de API/iOS antes de retirar estos planes.
