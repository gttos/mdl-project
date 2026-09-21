# Claude Code

Seguí [`AGENTS.md`](AGENTS.md). Es la instrucción canónica y común para todos los agentes que trabajan en `product/`.

Aplicá también el bloqueo de reglas inquebrantables de [`../AGENTS.md`](../AGENTS.md) §2: ante una contradicción o duda razonable con Product, detené la tarea completa, citá la regla y pedí al usuario que decida entre reabrirla o adaptar el trabajo.

Para comprobar código, aplicá íntegramente el protocolo de `codebase-memory-mcp` definido en `../AGENTS.md` §6. Claude Code no ejecuta `index_repository`.
