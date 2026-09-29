# Proing MCP — Documentación

Repositorio documental del proyecto **Proing MCP**.

El objetivo del proyecto es construir una primera implementación de un servidor MCP para exponer capacidades de consulta de información de Proing a agentes compatibles con Model Context Protocol, manteniendo la lógica de negocio y los cruces complejos fuera del agente.

## Principios iniciales

- El MCP se implementará inicialmente con **Node.js + TypeScript**.
- La primera versión se ejecutará localmente.
- El MCP consultará PostgreSQL como cliente, sin base de datos propia.
- Las consultas complejas, cruces y reglas de negocio deben resolverse preferiblemente mediante funciones o vistas en PostgreSQL.
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
└── 09-v1-definition.md

diagrams/
└── README.md
```

## Lectura recomendada

1. `docs/01-vision.md`
2. `docs/09-v1-definition.md`
3. `docs/02-architecture.md`
4. `docs/07-decisions.md`
5. `docs/08-current-status.md`

`docs/09-v1-definition.md` consolida la definición arquitectónica de la V1: alcance, stack, transportes MCP, estrategia PostgreSQL, seguridad, estructura del proyecto y criterio de evolución.

## Estado

Proyecto en fase de definición de la **versión 1 / MVP técnico**.
