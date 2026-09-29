# 02 — Arquitectura

## Arquitectura inicial

La primera versión se construirá con:

- **Node.js**
- **TypeScript**
- SDK de Model Context Protocol
- PostgreSQL como fuente de datos
- Ejecución local durante el MVP

El MCP no tendrá una base de datos propia en la primera versión.

## Flujo

```text
┌────────────────────────────┐
│ Agente / Cliente MCP       │
│ ChatGPT / Codex / Claude   │
│ Gemini / otro              │
└─────────────┬──────────────┘
              │ MCP
              ▼
┌────────────────────────────┐
│ Proing MCP                 │
│ Node.js + TypeScript       │
├────────────────────────────┤
│ Tools                      │
│ Resources                  │
│ Prompts                    │
├────────────────────────────┤
│ Validación                 │
│ Servicios por dominio      │
│ Repositorios / DB client   │
└─────────────┬──────────────┘
              │ SQL
              ▼
┌────────────────────────────┐
│ PostgreSQL                 │
├────────────────────────────┤
│ Vistas / funciones         │
│ preparadas para el MCP     │
├────────────────────────────┤
│ Modelo interno Proing      │
└────────────────────────────┘
```

## Organización lógica propuesta

```text
src/
├── server/
├── domains/
│   ├── operacion/
│   │   ├── tools/
│   │   ├── services/
│   │   └── repositories/
│   └── inventario/
├── resources/
├── prompts/
├── infrastructure/
│   └── database/
├── config/
└── shared/
```

La estructura definitiva se validará durante la implementación del primer Tool.

## Responsabilidades

### Agente

Debe:

- interpretar la intención del usuario;
- seleccionar el Tool correcto;
- construir los filtros permitidos;
- analizar el resultado.

No debe:

- construir joins arbitrarios;
- conocer todas las tablas;
- ejecutar SQL libre.

### MCP

Debe:

- publicar Tools con contratos claros;
- validar entradas;
- controlar acceso;
- transformar entradas y salidas cuando corresponda;
- registrar errores y operaciones relevantes;
- delegar la lógica de consulta.

### PostgreSQL

Debe resolver, cuando sea conveniente:

- joins complejos;
- agregaciones;
- normalización de información;
- reglas de consulta reutilizables.

## Estrategia de acceso a datos

Para escenarios con muchas tablas se priorizarán:

1. funciones PostgreSQL;
2. vistas;
3. consultas controladas dentro del repositorio del dominio.

No se expondrá el esquema completo al agente.

## Despliegue futuro

El MVP será local.

Posteriormente se evaluará ejecutar el MCP en infraestructura separada y liviana para no agregar carga innecesaria a los servidores actuales de aplicaciones.
