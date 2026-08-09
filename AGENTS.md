# AGENTS.md — Trabajo en el directorio product/

> **Ámbito:** `product/` — documentación compartida y canónica de MDL.  
> **Dirigido a:** Kimi (agente de razonamiento, análisis, planificación y coordinación transversal).  
> **Propósito:** Definir qué información vive en `product/`, cómo se organiza, cómo se mantiene y cómo se crean feature maestras.

## 1. Qué es product/

`product/` es la **fuente canónica de conocimiento compartido** del producto MDL. No contiene código. Todo documento nuevo en `product/` se escribe en **español**.

- **Documentación de producto:** flujos de usuario, reglas de negocio, comportamientos end-to-end, contratos de integración entre `api/` e `ios/`.
- **Documentación de arquitectura transversal:** visión general del sistema, límites de responsabilidad, servicios externos, fronteras de seguridad y privacidad.
- **Documentación legal/privacidad:** inventario de datos, tratamientos, políticas de retención, cumplimiento, riesgos. Redactada para revisión por abogado, DPO o especialista RGPD.
- **Decisiones transversales:** por qué se eligió una solución de producto o arquitectura que afecta a más de un proyecto.
- **Migraciones transversales:** cambios que requieren coordinación de datos, compatibilidad, rollout por fases o impacto entre versiones.
- **Planes cross-cutting:** feature maestras que orquestan el trabajo en `api/` e `ios/`.

## 2. Qué NO vive en product/

- **Documentación técnica de implementación:** specs de DDD, bounded contexts, controladores, servicios, tests, patrones de UI, networking local, almacenamiento local. Eso vive en `api/` o `ios/`.
- **Código, configuración, dependencias, CI/CD, Docker, Xcode, Laravel.** `product/` es documentación pura.
- **Documentación de `llm/`**. El microservicio de LLM es independiente. Su documentación vive en `llm/`.

## 3. Cuándo actualizar product/

| Cambio | Qué actualizar en product/ |
|---|---|
| Nueva regla de producto o negocio | `product/docs/product/` |
| Cambio de contrato entre cliente y backend | `product/docs/product/` + `product/docs/plans/active/` |
| Cambio en arquitectura transversal | `product/docs/architecture/` |
| Nueva decisión de producto/arquitectura | `product/docs/decisions/` |
| Cambio en tratamiento de datos, retención o privacidad | `product/docs/legal/` |
| Migración que afecte compatibilidad o datos | `product/docs/migrations/` |
| Feature cross-cutting que requiere coordinación | `product/docs/plans/active/<feature-slug>/` |
| Plan completado o cancelado | Mover de `active/` a `completed/` o `archived/` |

## 4. Sistema de planes cross-cutting

### 4.1 Feature maestra

Toda feature transversal debe tener una **feature maestra** en:

```
product/docs/plans/active/<feature-slug>/
```

El único `README.md` de la feature maestra debe permitir responder:

- ¿Qué cambia en el producto?
- ¿Qué reglas compartidas se ven afectadas?
- ¿Qué proyectos están involucrados?
- ¿Cómo deben interactuar funcionalmente los proyectos?
- ¿Qué orden general y dependencias existen?
- ¿Cuáles son los criterios de aceptación y el estado resumido?

### 4.2 Estructura de una feature maestra

```
product/docs/plans/active/<feature-slug>/
└── README.md              (producto, interacción, aceptación, orden y estado global)
```

No se crean documentos auxiliares dentro de la carpeta. Product define qué debe ocurrir; los detalles técnicos, operaciones, tests y riesgos de implementación viven en los planes locales.

### 4.3 Planes locales

Además de la feature maestra, Kimi debe crear **planes locales ejecutables** dentro de cada proyecto afectado:

```
api/docs/plans/active/<feature-slug>/
ios/docs/plans/active/<feature-slug>/
```

Cada carpeta local contiene únicamente `README.md`. Debe incluir la información necesaria para implementar ese lado sin duplicar la definición de producto:

- Objetivo local
- Fases numeradas (ej. API-01, API-02, IOS-01)
- Dependencia de otras fases
- Contrato esperado (qué endpoints, payloads, respuestas)
- Criterio de entrada
- Trabajo concreto de implementación
- Tests
- Validaciones manuales
- Criterio de salida
- Riesgos
- Bloqueo
- Siguiente fase permitida
- Referencia a la feature maestra en `product/`

### 4.4 Movimiento de planes entre estados

| Estado | Cuándo | Acción |
|---|---|---|
| **active** | La feature está en análisis, diseño o implementación. | Crear feature maestra + planes locales. |
| **completed** | La feature está terminada y sus criterios de aceptación se cumplieron. | Mover la feature maestra a `product/docs/plans/completed/<feature-slug>/`. Los planes locales pueden archivarse en los repos correspondientes. |
| **archived** | La feature fue cancelada, reemplazada o quedó obsoleta. | Mover la feature maestra a `product/docs/plans/archived/<feature-slug>/`. Conservar contexto suficiente para saber por qué fue archivada. |

## 5. Flujo de trabajo de Kimi en product/

1. Leer `product/AGENTS.md` (este documento).
2. Leer `product/docs/agents/README.md` (colaboración entre agentes).
3. Leer `product/docs/architecture/system-overview.md` (límites del sistema).
4. Analizar la feature cross-cutting: leer docs de producto, legal, decisiones relevantes.
5. Crear o actualizar la feature maestra en `product/docs/plans/active/<feature-slug>/`.
6. Crear planes locales en `api/docs/plans/active/` y `ios/docs/plans/active/` si aplica.
7. Presentar al usuario para aprobación.
8. Si el usuario aprueba, actualizar el estado dentro del `README.md` maestro y referenciar los planes locales.
9. Periódicamente, revisar progreso de planes locales y actualizar la feature maestra si hay bloqueos o cambios de scope.

## 6. Reglas no negociables

1. **No duplicar documentación de implementación.** Si un spec explica cómo funciona un Controller de Laravel o un ViewModel de Swift, vive en `api/` o `ios/`, no en `product/`.
2. **Todo Markdown nuevo en español.** Los documentos existentes en inglés se preservan en su idioma original.
3. **No crear documentación vacía.** No generes archivos de relleno. Crea solo cuando haya material real que documentar.
4. **La feature maestra es la fuente de verdad.** Si un plan local contradice la feature maestra, la feature maestra gana.
5. **Kimi no toca código.** Kimi analiza, planifica y documenta. La implementación la hacen los agentes locales (Antigravity, Codex) dentro de `api/` e `ios/`.
6. **Un archivo por nivel.** Cada plan activo usa un `README.md` maestro y un `README.md` por proyecto afectado; no se crean specs, matrices, prompts o reportes separados dentro de la carpeta del plan.

## 7. Referencia rápida de documentos canónicos

```
product/
├── README.md
├── AGENTS.md
└── docs/
    ├── agents/
    │   └── README.md          (colaboración entre agentes)
    ├── architecture/
    │   └── system-overview.md (visión general del sistema)
    ├── product/
    ├── legal/
    ├── decisions/
    ├── plans/
    │   ├── active/            (features en curso)
    │   ├── completed/         (features terminadas)
    │   └── archived/          (features canceladas u obsoletas)
    ├── migrations/
    ├── templates/
    └── archive/
```
