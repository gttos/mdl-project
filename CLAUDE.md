# Claude Code

Seguí [`AGENTS.md`](AGENTS.md). Es la instrucción canónica y común para todos los agentes que trabajan en `product/`.

Aplicá el protocolo de alcance y bloqueos de [`../AGENTS.md`](../AGENTS.md) §2 y [foco y hallazgos fuera de alcance](../AGENTS.md#foco-y-hallazgos-fuera-de-alcance) §3. Un conflicto necesario para ejecutar la tarea se eleva; un hallazgo lateral no la reemplaza. Consultá agrupadamente si el usuario quiere anotarlo en la lista única antes de escribirlo; no abras otra investigación, correctivo, fase o prompt por iniciativa propia.

Para comprobar código, aplicá íntegramente el protocolo de `codebase-memory-mcp` definido en `../AGENTS.md` §6. Claude Code no ejecuta `index_repository`.
