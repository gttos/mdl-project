# .agents/AGENTS.md — Orquestación de agentes en el workspace MDL

> **Rol:** Punto de entrada para agentes de IA que operan en el workspace MDL completo.  
> **Ubicación:** `product/.agents/AGENTS.md`  
> **Propósito:** Determinar si una tarea requiere orquestación entre repos, y si es así, cómo se coordina el flujo entre Antigravity y Kimi antes de tocar código en `ios/` o `api/`.

## 1. Identificación de alcance

Responde antes de actuar:

| Pregunta | Si la respuesta es sí |
|---|---|
| ¿La tarea afecta solo un repo (`ios` o `api`)? | Saltar directamente al AGENTS.md de ese repo (`ios/.agents/AGENTS.md` o `api/.agents/AGENTS.md`). |
| ¿La tarea afecta contratos entre cliente y backend? | Leer este documento completo + `product/docs/` + luego repos afectados. |
| ¿Es un cambio de producto, privacidad, legal, seguridad transversal o migración? | Leer este documento completo + `product/docs/` + luego repos afectados. |
| ¿Se requiere coordinar fases entre backend y app? | Leer este documento completo + `product/docs/plans/active/`. |

## 2. Agentes y flujo de trabajo

```
[Usuario] → [Identificar alcance] → ¿Transversal?
  │                              │
  No → Saltar al repo afectado   Sí → [Antigravity] → [Aprobación] → [Kimi] → [Revisión]
         (ios/.agents/ o             (product/.agents/ANTIGRAVITY.md)   (product/.agents/KIMI.md)
          api/.agents/)
```

| Agente | Punto de entrada | Rol |
|---|---|---|
| **Antigravity** | `product/.agents/ANTIGRAVITY.md` | Diseño, requerimientos, arquitectura transversal. No toca código. |
| **Kimi** | `product/.agents/KIMI.md` | Implementación transversal. Escribe código, tests, coordina despliegues. |

## 3. Protocolo de lectura obligatorio (tareas transversales)

1. Leer `product/.agents/AGENTS.md` (este documento).
2. Leer `product/AGENTS.md` (visión general del workspace).
3. Leer `product/docs/agents/README.md` (colaboración entre agentes).
4. Leer `product/docs/architecture/system-overview.md` (límites del sistema).
5. Leer documentos específicos de `product/docs/` según el área:
   - `product/docs/product/` — flujos de producto
   - `product/docs/legal/` — privacidad, retención, compliance
   - `product/docs/decisions/` — decisiones arquitectónicas transversales
   - `product/docs/plans/active/` — planes activos
6. Identificar repos afectados (`ios/` y/o `api/`).
7. Leer el punto de entrada de cada repo afectado:
   - `ios/.agents/AGENTS.md` y `ios/docs/AGENTS.md`
   - `api/.agents/AGENTS.md` y `api/AGENTS.md`
8. Seguir el routing de specs en cada repo para cargar solo documentos relevantes.

## 4. MCP obligatorio

**Codebase Memory MCP** está disponible en todos los repos (`ios/.kimi-code/mcp.json`, `api/.kimi-code/mcp.json`). Tools: `search_graph`, `trace_path`, `query_graph`.

- **Antigravity:** Usa MCP en el repo donde necesita entender impacto arquitectónico antes de proponer diseños.
- **Kimi:** Usa MCP obligatoriamente en cada repo antes de tocar código.
- Si MCP falla, declararlo explícitamente y usar fallback manual (grep + read).

## 5. Orquestación de fases (ejemplo: Vault)

| Fase | Repos activo | Documento de coordinación |
|---|---|---|
| Diseño transversal | `product/` | `product/docs/plans/active/vault-e2ee/README.md` |
| Diseño backend | `api/` | `api/.agents/ANTIGRAVITY.md` + specs locales |
| Diseño iOS | `ios/` | `ios/.agents/ANTIGRAVITY.md` + specs locales |
| Implementación backend | `api/` | `api/.agents/KIMI.md` + specs aprobados |
| Implementación iOS | `ios/` | `ios/.agents/KIMI.md` + specs aprobados |
| Validación legal | `product/` | `product/docs/legal/` |

## 6. Qué no hacer

- No implementar código directamente desde el nivel del workspace sin pasar por el repo afectado.
- No ignorar el routing de specs de cada repo.
- No omitir MCP en ningún repo.
- No crear specs en `product/` sin validar que no contradigan los specs locales de `ios/` o `api/`.

## 7. Referencias rápidas

- Orquestación general: `product/.agents/AGENTS.md` (este archivo)
- Visión general del workspace: `product/AGENTS.md`
- Colaboración agentes: `product/docs/agents/README.md`
- Arquitectura transversal: `product/docs/architecture/system-overview.md`
- iOS: `ios/.agents/AGENTS.md` y `ios/docs/AGENTS.md`
- API: `api/.agents/AGENTS.md` y `api/AGENTS.md`
