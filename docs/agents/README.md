# Agentes de IA en MDL — Roles y Colaboración

> **Ámbito:** Producto completo (iOS + Backend + LLM).  
> **Audiencia:** Agentes de IA que trabajen en el workspace MDL.  
> **Propósito:** Definir quién hace qué, en qué orden, y cómo usan MCP para no desperdiciar contexto.

## 1. Tres roles, tres responsabilidades

En MDL operan tres roles de agentes de IA con responsabilidades diferenciadas. **No son intercambiables.**

| Agente | Rol | Ámbito | Qué hace | Qué NO hace | Documento de entrada |
|---|---|---|---|---|---|
| **Kimi** | Razonamiento, análisis, planificación y coordinación transversal | Workspace raíz `mdl/` | Analiza features cross-cutting. Mantiene un único `README.md` maestro y un único `README.md` local por proyecto afectado. Define producto, interacción general, orden y aceptación. | **No implementa código.** No toca tests. No modifica configuración. No ejecuta migraciones. No marca fases como completadas sin evidencia. | `mdl/AGENTS.md` |
| **Antigravity** | Producto y arquitectura (local) | Dentro de `api/` o `ios/` | Traduce decisiones de producto en requerimientos, specs y diseños para un solo repo. Valida impacto arquitectónico con MCP. | **No escribe código de producción.** No implementa tests. No modifica archivos fuente. | `api/.agents/ANTIGRAVITY.md` o `ios/.agents/ANTIGRAVITY.md` |
| **Codex / Kimi local** | Implementación técnica (local) | Dentro de `api/` o `ios/` | Ejecuta una fase concreta del plan local. Escribe código, tests, docs locales. Reporta estado. | **No avanza fases por iniciativa propia.** No redefina reglas de producto. No modifica `product/`. No modifica el otro repo. | `api/.agents/KIMI.md` o `ios/.agents/KIMI.md` |

## 2. Flujo de trabajo completo

```
[Usuario describe feature cross-cutting]
    ↓
[Kimi desde mdl/] → Analiza, lee docs, define arquitectura
    ↓
Kimi crea feature maestra en product/docs/plans/active/<feature-slug>/
Kimi crea planes locales en api/docs/plans/active/ y ios/docs/plans/active/
    ↓
[Usuario aprueba feature maestra y planes]
    ↓
[Usuario abre agente implementador en api/ o ios/]
    ↓
Agente implementador lee AGENTS.md local + plan local
    ↓
Agente implementador ejecuta UNA fase explícita (ej. API-02, IOS-03)
    ↓
Agente implementador reporta estado en el plan local
    ↓
[Usuario revisa progreso]
    ↓
[Kimi revisa progreso transversal]
    ↓
Siguiente fase
```

### 2.1 Kimi: fase de planificación transversal

1. El usuario describe una feature cross-cutting desde la raíz del workspace.
2. Kimi lee `mdl/AGENTS.md`, `product/AGENTS.md`, y `product/docs/agents/README.md`.
3. Kimi lee `product/docs/architecture/system-overview.md` y docs relevantes de `product/docs/`.
4. Kimi lee progresivamente los specs locales de `api/` e `ios/` según el routing de cada repo.
5. Kimi usa **Codebase Memory MCP** (`search_graph`, `trace_path`) en cada repo para entender el impacto sin cargar todo el código.
6. Kimi valida findings de MCP contra docs y código fuente.
7. Kimi crea o actualiza:
   - `product/docs/plans/active/<feature-slug>/README.md`
   - `api/docs/plans/active/<feature-slug>/README.md` y/o `ios/docs/plans/active/<feature-slug>/README.md`
   - Docs compartidas si afectan producto, legal, arquitectura o decisiones
8. Kimi presenta al usuario para aprobación.
9. Kimi revisa periódicamente el progreso y actualiza la feature maestra si hay bloqueos o cambios de scope.

### 2.2 Antigravity: fase de diseño local (cuando no hay plan global)

1. El usuario describe una tarea local dentro de un solo repo.
2. Antigravity lee `api/AGENTS.md` o `ios/docs/AGENTS.md`, y su routing local.
3. Antigravity lee los specs locales identificados por el routing.
4. Antigravity usa MCP para entender el impacto arquitectónico.
5. Antigravity crea `docs/features/[nombre]/requirements.md` y `design.md`.
6. Antigravity presenta al usuario para aprobación.

### 2.3 Codex / Kimi local: fase de implementación por fases

1. El usuario indica una fase concreta a implementar (ej. "Implementa API-02").
2. El agente lee el `AGENTS.md` local y el routing local.
3. El agente lee el plan local en `docs/plans/active/<feature-slug>/`.
4. El agente lee la feature maestra si necesita entender contratos transversales.
5. El agente usa MCP obligatoriamente para impact analysis de código.
6. El agente aplica Imitation Learning (regla de los 2 ejemplos) antes de crear código nuevo.
7. El agente implementa **únicamente la fase indicada**. No avanza a fases posteriores.
8. El agente reporta: estado, archivos modificados, tests, validaciones, bloqueos, riesgos, siguiente fase permitida.

## 3. Uso de Codebase Memory MCP — obligatorio, no opcional

Todos los agentes deben usar MCP como herramienta primaria para entender el sistema sin cargar todo el código en contexto. Esto ahorra tokens y reduce alucinaciones.

### 3.1 Tools disponibles

| Tool | Uso | Cuándo usarlo |
|---|---|---|
| `search_graph` | Buscar símbolos, clases, funciones, módulos en el grafo de código. | Cuando necesitas encontrar dónde vive algo sin grepear todo el repo. |
| `trace_path` | Seguir un path de llamadas o dependencias entre módulos. | Cuando necesitas entender cómo se conecta A con B, o qué se rompería si cambias C. |
| `query_graph` | Consultar relaciones específicas del grafo (ej. "qué controladores usan este servicio"). | Cuando necesitas validar un supuesto arquitectónico. |

### 3.2 Protocolo obligatorio para implementación (Codex / Kimi local)

Antes de modificar cualquier archivo de código:

1. **Ejecutar `search_graph`** para localizar el módulo o símbolo que vas a modificar.
2. **Ejecutar `trace_path`** desde ese símbolo hacia sus consumidores y dependencias.
3. **Ejecutar `query_graph`** si necesitas validar relaciones específicas (ej. "¿qué tests cubren este método?").
4. **Validar findings contra código fuente:** MCP puede estar desactualizado o indexar archivos de instrucciones. No confíes ciegamente. Lee los archivos relevantes que MCP identificó.
5. **Si MCP falla o no responde:** usar fallback de búsqueda manual (grep + read de archivos relevantes). Documentar que MCP no estuvo disponible.

### 3.3 Protocolo para Kimi (planificación transversal) y Antigravity (diseño)

Antes de proponer un diseño o plan:

1. **Ejecutar `search_graph`** para entender qué módulos existen en el área afectada.
2. **Ejecutar `trace_path`** para mapear el impacto de un cambio propuesto.
3. **Validar findings contra specs y documentación:** MCP no indexa archivos de instrucciones (`.md` de specs). Lee los docs manualmente después del análisis MCP.
4. **Si MCP falla:** usar búsqueda manual en repo + lectura de specs.

## 4. Límites de MCP conocidos

- **No indexa archivos de instrucciones:** Los archivos `.md` de specs, AGENTS.md y docs no aparecen como nodos en el grafo. MCP sirve para código; los specs se leen manualmente.
- **Puede estar desactualizado:** Si el código cambió recientemente, el grafo puede no reflejarlo. Siempre valida.
- **No reemplaza el juicio arquitectónico:** MCP te dice dónde está el código; no te dice si un cambio es correcto de producto.

## 5. Qué leer según tu rol

### Kimi (planificación transversal) lee primero:
- `mdl/AGENTS.md` (punto de entrada del workspace)
- `product/AGENTS.md` (propiedad documental)
- `product/docs/agents/README.md` (este documento)
- `product/docs/architecture/system-overview.md` (límites del sistema)
- Docs relevantes de `product/docs/` (producto, legal, decisiones, planes)
- `api/AGENTS.md` y `api/docs/agent-routing.md` si afecta backend
- `ios/docs/AGENTS.md` y `ios/docs/routing/agent-routing.md` si afecta iOS

### Antigravity (diseño local) lee primero:
- `api/AGENTS.md` o `ios/docs/AGENTS.md` (punto de entrada del repo)
- Routing del proyecto (`agent-routing.md`) para identificar specs relevantes
- Specs identificados por el routing
- `product/docs/` si la tarea es transversal

### Codex / Kimi local (implementación) lee primero:
- `api/AGENTS.md` o `ios/docs/AGENTS.md` (punto de entrada del repo)
- El documento de agente específico del proyecto (`.agents/KIMI.md`)
- Routing del proyecto para identificar specs relevantes
- Plan local en `docs/plans/active/<feature-slug>/` (si existe)
- Feature maestra en `product/docs/plans/active/<feature-slug>/` (si aplica)
- Specs de feature aprobados
- **Luego:** MCP obligatorio para impact analysis de código

## 6. Reglas no negociables para todos los agentes

1. **No inventar reglas de producto:** Si un spec dice una cosa y el código otra, documentar el conflicto. No resolverlo por intuición.
2. **Kimi no toca código:** Kimi analiza, planifica y documenta. Nunca implementa ni modifica código de producción.
3. **No escribir código sin specs aprobados:** Los implementadores no implementan sin `requirements.md` y `design.md` aprobados, o sin un plan local activo aprobado.
4. **No avanzar fases por iniciativa propia:** Los implementadores ejecutan únicamente la fase indicada explícitamente por el usuario.
5. **No duplicar reglas compartidas en docs locales:** Si una regla vive en `product`, referenciarla. No copiarla.
6. **Actualizar documentación en el mismo commit:** Si el código cambia, los specs afectados se actualizan en el mismo commit.
7. **Preservar el idioma de los documentos existentes:** Los docs nuevos en `product` van en español. Los docs locales existentes en inglés se preservan.
8. **No fragmentar planes:** Una feature usa un solo `README.md` en Product y uno en cada proyecto afectado.

## 7. Documentos específicos por agente y proyecto

- `mdl/AGENTS.md` — Instrucciones para Kimi desde la raíz del workspace
- `product/AGENTS.md` — Instrucciones para trabajo en documentación compartida
- `product/.agents/ANTIGRAVITY.md` — Instrucciones para Kimi diseñando en `product/`
- `product/.agents/KIMI.md` — Instrucciones para Kimi coordinando en `product/`
- `api/.agents/ANTIGRAVITY.md` — Instrucciones para Antigravity en API
- `api/.agents/KIMI.md` — Instrucciones para implementadores en API
- `ios/.agents/ANTIGRAVITY.md` — Instrucciones para Antigravity en iOS
- `ios/.agents/KIMI.md` — Instrucciones para implementadores en iOS
- `product/.agents/CODEX.md` — Instrucciones para Codex transversal
- `api/.codex/AGENTS.md` — Instrucciones para Codex en API
- `ios/.codex/AGENTS.md` — Instrucciones para Codex en iOS

## 8. Fallback cuando MCP no está disponible

Si el servidor MCP no responde (timeout, error de conexión, comando no encontrado):

1. **Declarar explícitamente** que MCP no está disponible en esta sesión.
2. **Usar búsqueda manual:** `grep` / `find` / `rg` para localizar archivos relevantes.
3. **Leer archivos clave:** controllers, services, models, tests del área afectada.
4. **Construir mental map** de dependencias leyendo imports y llamadas.
5. **Documentar** en el reporte final que se usó fallback manual.

---

> **Nota:** Este documento es el punto de entrada para entender la colaboración entre agentes. Para instrucciones específicas de tu rol en un contexto concreto, lee el AGENTS.md de tu nivel (raíz, producto, o repo local).
