# product — Documentación compartida y canónica de MDL

## Qué es este directorio

`product` contiene la documentación compartida y canónica del producto **My Ducking Life (MDL)**.

- `ios` y `api` siguen siendo proyectos y repositorios Git independientes.
- Este directorio **no** es un monorepo. No contiene código fuente, tests ni configuraciones de build.
- Su único propósito es centralizar el conocimiento que pertenece al producto completo: flujos end-to-end, contratos entre cliente y backend, arquitectura transversal, privacidad, legal, decisiones compartidas, planes y migraciones.

## Qué información vive aquí

- **Comportamiento compartido de producto:** reglas de negocio, flujos de autenticación, ciclos de vida de entidades, contratos de integración.
- **Arquitectura transversal:** visión general del sistema, límites de responsabilidad, servicios externos, fronteras de seguridad y privacidad.
- **Privacidad, legal y retención:** inventario de datos, tratamientos, políticas de retención, cumplimiento y riesgos que afectan a todo el producto.
- **Decisiones de producto y arquitectura:** por qué se eligió una solución transversal (por ejemplo, el modelo E2EE del Vault).
- **Planes y migraciones:** cambios que afectan a más de un proyecto, con fases, coordinación y compatibilidad.
- **Conflictos y lagunas:** puntos donde la documentación existente es contradictoria o incompleta, pendientes de revisión humana.

## Qué información permanece en los repositorios técnicos

- **ios:** documentación específica de implementación iOS (SwiftUI, arquitectura de UI, networking local, almacenamiento local, Keychain, manejo de estado, testing iOS).
- **api:** documentación específica de implementación backend (Laravel, DDD, bounded contexts, jobs, colas, políticas, specs de arquitectura API).

Cada proyecto conserva su propio sistema de instrucciones para agentes (`AGENTS.md`, `agent-routing.md`), su estructura de specs y su worklog de cambios locales.

## Cómo se documentan los cambios transversales

1. Si un cambio afecta comportamiento de producto, contratos, integración o más de un proyecto, se crea o actualiza un documento en `product/docs/`.
2. Si el cambio requiere coordinación por fases, se crea un plan en `product/docs/plans/active/`.
3. Cuando el plan finaliza, se mueve a `product/docs/plans/completed/`.
4. Los repositorios técnicos actualizan solo su documentación local afectada y referencian la fuente canónica en `product`.

## Estructura

```
product/
├── README.md                         (este archivo)
├── AGENTS.md                         (punto de entrada para agentes transversales)
└── docs/
    ├── architecture/
    │   └── system-overview.md        (visión general del sistema MDL)
    ├── product/
    │   └── flujos compartidos (autenticación, media, timeline, Vault)
    ├── legal/
    │   └── privacidad, retención, tratamientos, compliance
    ├── decisions/
    │   └── registros de decisiones transversales
    ├── plans/
    │   ├── active/                   (planes en curso)
    │   ├── completed/                (planes terminados)
    │   └── archived/                 (planes cancelados o obsoletos)
    ├── migrations/
    │   └── migraciones transversales (Vault, compatibilidad API/iOS)
    ├── templates/
    │   └── plantillas reutilizables para planes y decisiones
    └── archive/
        └── material histórico o reemplazado
```

## Idioma

Todo documento nuevo en `product` está escrito en **español**.
Los documentos existentes en inglés que se referencian desde aquí se preservan en su idioma original.
