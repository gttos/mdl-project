# CODEX.md — Instrucciones para Codex en el workspace MDL

> **Rol:** Punto de entrada para Codex (OpenAI) cuando opera en el workspace MDL.  
> **Propósito:** Orquestar qué repo tocar, qué documentación leer, y cómo usar MCP para ahorrar tokens.

## 1. Identificación de alcance

Antes de escribir código:

| Pregunta | Acción |
|---|---|
| ¿La tarea afecta solo `ios/`? | Leer `ios/.codex/AGENTS.md` y seguir instrucciones iOS. |
| ¿La tarea afecta solo `api/`? | Leer `api/.codex/AGENTS.md` y seguir instrucciones API. |
| ¿Afecta ambos o es transversal? | Leer este documento + `product/.agents/AGENTS.md` + `product/docs/` + repos afectados. |

## 2. Flujo de trabajo (transversal)

1. **Leer `product/.agents/AGENTS.md`** — orquestación general.
2. **Leer `product/AGENTS.md`** — visión del workspace.
3. **Leer `product/docs/agents/README.md`** — colaboración entre agentes.
4. **Leer specs aprobados** en `product/docs/` (decisiones, planes, producto, legal).
5. **Identificar repos afectados** (`ios/` y/o `api/`).
6. **Leer puntos de entrada de cada repo**:
   - `ios/.codex/AGENTS.md` para iOS
   - `api/.codex/AGENTS.md` para API
7. **Usar MCP** (`search_graph`, `trace_path`, `query_graph`) en cada repo antes de tocar código.
8. **Validar findings** contra código fuente y documentación.

## 3. MCP obligatorio

- **MCP está activo en todos los repos.** Configuración:
  - `ios/.kimi-code/mcp.json`
  - `api/.kimi-code/mcp.json`
- Tools: `search_graph`, `trace_path`, `query_graph`.
- Usar MCP para entender el sistema sin cargar todo el código en contexto.
- Si MCP falla, declararlo explícitamente y usar fallback manual (grep + read).

## 4. Reglas

- No escribir código sin leer las instrucciones del repo afectado.
- No ignorar specs aprobados en `product/docs/`.
- No omitir MCP.
- No duplicar reglas compartidas en documentación local. Usar referencias.

## 5. Referencias

- `product/.agents/AGENTS.md` — orquestación general
- `product/docs/agents/README.md` — colaboración entre agentes
- `ios/.codex/AGENTS.md` — instrucciones específicas iOS
- `api/.codex/AGENTS.md` — instrucciones específicas API
