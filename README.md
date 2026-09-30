# Proing MCP — Documentación

Repositorio documental del proyecto **Proing MCP**.

El objetivo del proyecto es construir una primera implementación de un servidor MCP para exponer capacidades de consulta de información de Proing a agentes compatibles con Model Context Protocol, manteniendo la lógica de negocio y los cruces complejos fuera del agente.

## Principios iniciales

- El MCP se implementará con **Node.js + TypeScript**.
- La primera versión se ejecutará localmente.
- El MCP consultará PostgreSQL directamente para capacidades de lectura/reporting.
- Las consultas complejas, cruces y reglas se resolverán mediante Views, Functions o consultas controladas.
- Las acciones de negocio futuras utilizarán preferentemente APIs/servicios del dominio.
- El agente no debe conocer ni manipular directamente el modelo relacional interno de Proing.
- La evolución del MCP será por dominios: operación, inventario y otros que se definan posteriormente.
- Código y documentación podrán vivir inicialmente en repositorios separados y consolidarse más adelante en GitLab.

## Estructura documental

```text
docs/
├── 01-vision.md
├── 02-architecture.md
├── 03-tools.md
├── 04-resources.md
├── 05-prompts.md
├── 06-security.md
├── 07-decisions.md
├── 08-current-status.md
└── 09-implementation-plan.md

diagrams/
└── README.md
```

## Responsabilidad de cada documento

- `01-vision.md`: propósito y visión del producto.
- `02-architecture.md`: arquitectura técnica, integración, transportes, estructura y evolución.
- `03-tools.md`: criterios y contratos de Tools.
- `04-resources.md`: estrategia futura de Resources.
- `05-prompts.md`: estrategia futura de Prompts.
- `06-security.md`: controles, permisos, secretos, límites y protección de datos.
- `07-decisions.md`: decisiones arquitectónicas aprobadas.
- `08-current-status.md`: estado, pendientes y próximo hito.
- `09-implementation-plan.md`: ejecución del MVP por bloques, con validaciones y criterios de cierre.

## Lectura recomendada

1. `docs/01-vision.md`
2. `docs/02-architecture.md`
3. `docs/07-decisions.md`
4. `docs/06-security.md`
5. `docs/08-current-status.md`
6. `docs/09-implementation-plan.md`

## Estado

Proyecto en fase de definición final del **MVP técnico / V1**.

El siguiente hito es ejecutar el MVP siguiendo `docs/09-implementation-plan.md`, empezando por el Bloque 0.
