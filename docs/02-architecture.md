# 02 — Arquitectura

## Objetivo

Definir la arquitectura base del **Proing MCP** para la V1 y su evolución hacia múltiples dominios.

La V1 será deliberadamente pequeña: un único servidor MCP, un primer dominio de **Operación** y una primera Tool orientada a consultar histórico de vehículos.

El diseño debe permitir crecer posteriormente hacia Operación, Inventario, Personal y otros dominios sin reorganizar el proyecto desde cero.

---

## Principios arquitectónicos

### Organización por dominio

El MCP no se estructura alrededor de tablas, sino alrededor de capacidades de negocio.

```text
PROING MCP
│
├── operacion.*
├── inventario.*
├── personal.*
└── otros dominios futuros
```

Internamente:

```text
domains/
├── operacion/
├── inventario/
└── personal/
```

La cantidad de Tools por sí sola no obliga a crear múltiples servidores MCP. Se evaluará separar dominios únicamente cuando existan fronteras reales de seguridad, infraestructura, equipo, despliegue o escala.

### El agente no conoce el modelo físico

El agente no debe conocer:

- nombres de tablas;
- claves internas;
- JOIN;
- relaciones;
- SQL;
- columnas técnicas;
- reglas de persistencia.

El agente trabaja con conceptos de negocio y contratos explícitos.

Ejemplo:

```text
Usuario
  ↓
"Muéstrame el histórico del vehículo ABC123"
  ↓
Agente
  ↓
operacion.consultar_historico_vehiculos
```

No se expondrá una Tool genérica del tipo `execute_sql`.

---

## Arquitectura general V1

```text
                 AGENTE / CLIENTE
          Claude / Gemini / ChatGPT
                       │
                       │ MCP
                       ▼
              ┌─────────────────┐
              │   PROING MCP    │
              │                 │
              │ Node.js         │
              │ TypeScript      │
              │ MCP SDK         │
              └────────┬────────┘
                       │
                  módulo dominio
                       │
                       ▼
               Cliente PostgreSQL
                       │
                       ▼
              PostgreSQL Proing
                       │
                       ▼
                 schema mcp
                       │
                       ▼
              Views / Functions
                       │
                       ▼
              Tablas operacionales
```

Para la V1 no se creará una API REST intermedia únicamente para envolver la consulta histórica.

---

## Estrategia de integración

El MCP puede consumir diferentes tipos de backend. Para Proing se adopta inicialmente un enfoque híbrido.

### Lectura y reporting

Preferentemente:

```text
MCP
 ↓
PostgreSQL
 ↓
Views / Functions controladas
 ↓
Tablas operacionales
```

Adecuado para:

- consultas;
- reporting;
- históricos;
- agregaciones;
- análisis de información.

### Acciones y procesos de negocio

Preferentemente:

```text
MCP
 ↓
API / Servicio del dominio
 ↓
reglas de negocio
 ↓
PostgreSQL
```

Adecuado para:

- registrar información;
- modificar estados;
- ejecutar procesos;
- transacciones;
- lógica ya utilizada por otros sistemas.

Ejemplo futuro:

```text
consultar productividad      → PostgreSQL
consultar histórico          → PostgreSQL
consultar consumo            → PostgreSQL

registrar salida inventario  → API Inventario
cerrar orden                 → API Operación
```

---

## Capa de datos para MCP

Se propone reservar un schema PostgreSQL para las superficies de datos destinadas al MCP:

```text
mcp
```

Ejemplo:

```text
PostgreSQL
│
├── schemas/tablas operacionales
│
└── mcp
    ├── vw_historico_vehiculos
    ├── vw_ordenes
    ├── vw_productividad
    ├── fn_productividad(...)
    └── fn_consumo_materiales(...)
```

Esta capa traduce:

```text
MODELO FÍSICO PROING
        ↓
MODELO DE NEGOCIO
        ↓
MCP TOOL
```

El usuario y el agente no necesitan conocer cómo están relacionadas las tablas originales.

---

## Views, Functions y consultas controladas

### View

Se utilizará cuando exista una representación reutilizable y relativamente estable.

Ejemplo:

```text
mcp.vw_historico_vehiculos
```

El MCP aplica filtros de negocio sobre la vista.

### Function

Se utilizará cuando la consulta tenga:

- múltiples JOIN;
- agregaciones;
- cálculos;
- reglas;
- filtros complejos;
- construcción específica de un reporte.

Ejemplo:

```text
mcp.fn_productividad(
    fecha_inicio,
    fecha_fin,
    contrato
)
```

### Consulta controlada en el repositorio

También se permite una consulta SQL parametrizada dentro de la capa de datos del dominio cuando crear una View o Function no aporte reutilización.

La decisión entre View, Function y SQL controlado es técnica; el contrato MCP no debe depender del modelo físico.

### Catálogos de dominio en el MCP

Para catálogos pequeños, estables y exclusivos del MCP, la V1 permite mantenerlos como archivos versionados dentro del artefacto.

El primer caso será el catálogo de eventos de vehículos:

```text
src/
└── domains/
    └── operacion/
        └── catalogs/
            └── eventos-vehiculo.json
```

El histórico conservará el valor original del proveedor y la capa de dominio del MCP lo normalizará usando este catálogo.

```text
PostgreSQL
   ↓
evento original
   ↓
Repository MCP
   ↓
eventos-vehiculo.json
   ↓
codigo / nombre / descripcion Proing
   ↓
respuesta MCP
```

No se requiere una tabla catálogo PostgreSQL para este caso en el MVP. Si en el futuro el catálogo necesita edición dinámica, administración por usuarios o reutilización por otros sistemas, se reevaluará su persistencia.

---

## Responsabilidades

### Agente

Debe:

- interpretar la intención del usuario;
- seleccionar la Tool adecuada;
- proporcionar filtros permitidos;
- analizar el resultado.

No debe:

- construir JOIN arbitrarios;
- descubrir el modelo relacional;
- ejecutar SQL libre.

### MCP

Debe:

- publicar Tools con contratos claros;
- validar entradas;
- validar límites;
- invocar la capa de datos o servicio;
- controlar errores;
- normalizar resultados;
- registrar operaciones relevantes.

### PostgreSQL / servicios de dominio

Deben resolver, según corresponda:

- JOIN complejos;
- agregaciones;
- reglas reutilizables;
- normalización;
- procesos de negocio.

---

## Stack tecnológico V1

```text
Runtime:            Node.js LTS
Lenguaje:           TypeScript
MCP:                MCP TypeScript SDK v2
Servidor MCP:       @modelcontextprotocol/server
Validación:         Zod
PostgreSQL:         pg
Gestor paquetes:    npm
Módulos:            ESM
Control versiones:  Git
Repositorio código: GitLab Proing
```

---

## Transportes MCP

El núcleo del servidor no debe depender del transporte.

```text
                 MCP CORE
                    │
            createProingServer()
                    │
            ┌───────┴────────┐
            │                │
          stdio       Streamable HTTP
```

### stdio

Será el transporte principal durante el desarrollo local.

```text
Cliente MCP local
       ↓
     stdio
       ↓
   Proing MCP
```

### Streamable HTTP

El mismo servidor podrá exponerse por:

```text
http://localhost:<puerto>/mcp
```

y posteriormente en AWS mediante HTTPS.

La lógica de las Tools debe ser la misma independientemente del transporte.

---

## Compatibilidad con agentes

El servidor no tendrá lógica específica para Claude, Gemini o ChatGPT.

```text
             PROING MCP
                 ▲
       ┌─────────┼─────────┐
       │         │         │
     Claude    Gemini   ChatGPT
```

Cada cliente MCP decide cómo descubre, presenta y ejecuta las Tools.

Durante desarrollo local se priorizarán clientes compatibles con `stdio` o Streamable HTTP. Para clientes que requieran acceso remoto se publicará posteriormente el endpoint MCP de forma segura.

---

## Desarrollo local

La primera iteración deberá ejecutarse desde la máquina del desarrollador.

```text
Mac / PC
│
├── Node.js
├── Proing MCP
└── conexión autorizada a PostgreSQL
```

Flujo esperado:

```text
desarrollo local
      ↓
pruebas MCP
      ↓
prueba con agentes
      ↓
despliegue AWS
```

La EC2 definitiva no es requisito para construir ni validar la V1.

---

## Acceso PostgreSQL

El MCP contará con un único pool PostgreSQL compartido.

```text
domains/
   │
   ├── operacion
   ├── inventario futuro
   └── ...
          │
          ▼
infrastructure/database/postgres
          │
          ▼
       Pool
          │
          ▼
     PostgreSQL
```

No se abrirá una conexión nueva por cada Tool.

Las consultas serán parametrizadas y controladas por el código o por funciones/vistas autorizadas.

---

## Estructura inicial del proyecto

```text
proing-mcp/
│
├── src/
│   ├── server/
│   │   └── create-server.ts
│   │
│   ├── transports/
│   │   ├── stdio.ts
│   │   └── http.ts
│   │
│   ├── domains/
│   │   └── operacion/
│   │       ├── tools/
│   │       │   └── vehicle-history.tool.ts
│   │       ├── contracts/
│   │       │   └── vehicle-history.contract.ts
│   │       ├── services/
│   │       │   └── vehicle-history.service.ts
│   │       ├── repositories/
│   │       │   ├── vehicle-history.repository.ts
│   │       │   └── vehicle-history.cursor.ts
│   │       └── catalogs/
│   │           ├── eventos-vehiculo.json
│   │           └── vehicle-events.catalog.ts
│   │
│   ├── infrastructure/
│   │   └── database/
│   │       └── postgres.ts
│   │
│   ├── config/
│   │   └── env.ts
│   │
│   └── index.ts
│
├── database/
│   ├── views/
│   │   └── vw_historico_vehiculos.sql
│   └── functions/
│
├── docs/
├── tests/
├── .env.example
├── package.json
├── tsconfig.json
└── README.md
```

La estructura debe mantenerse pequeña. No se incorporarán capas adicionales sin una necesidad concreta.

### Regla de responsabilidad por capa

Cuando la complejidad de una Tool lo justifique, se aplicará la siguiente separación:

```text
Transport
   ↓
MCP Server
   ↓
Tool
   ↓
Contract
   ↓
Service
   ↓
Repository
   ↓
PostgreSQL / API
```

Responsabilidades:

- **Tool**: frontera MCP, metadata, registro y traducción de errores al protocolo.
- **Contract**: schemas Zod de entrada/salida y tipos derivados del contrato público.
- **Service**: orquestación del caso de uso y reglas de aplicación.
- **Repository**: acceso a datos, SQL parametrizado y paginación.
- **Catalog**: normalización de conocimiento estático del dominio.
- **Transport**: mecanismo de conexión MCP, sin lógica de dominio.

No todas las Tools requieren obligatoriamente un Service. La capa se introduce cuando existe lógica de aplicación suficiente para justificarla.

Se evitarán carpetas genéricas como `helpers/`, `utils/` o `common/` cuando oculten responsabilidades. Los componentes compartidos solo se crearán cuando exista reutilización real.

---

## Configuración

La aplicación utilizará configuración externa al código.

Variables conceptuales:

```text
MCP_NAME
MCP_VERSION

DATABASE_HOST
DATABASE_PORT
DATABASE_NAME
DATABASE_USER
DATABASE_PASSWORD
DATABASE_POOL_MAX

APP_TIMEZONE=America/Bogota
HTTP_PORT
```

`.env` se utilizará durante desarrollo local para credenciales y configuración sensible y no se versionará. El repositorio incluirá únicamente `.env.example` sin secretos.

---

## Evolución esperada

### V1

```text
PROING MCP
└── operacion.consultar_historico_vehiculos
```

### Futuro

```text
PROING MCP
│
├── operacion.*
├── inventario.*
├── personal.*
└── otros dominios
```

Se evaluará separar servidores MCP únicamente si aparecen necesidades reales de aislamiento.

---

## Despliegue futuro

La primera versión se desarrolla localmente.

Posteriormente:

```text
Clientes autorizados
        │
        ▼
https://mcp.proing.com.co/mcp
        │
        ▼
     EC2 MCP
        │
        ├── PostgreSQL Proing
        └── APIs/servicios Proing
```

El MCP se desplegará en una instancia separada de la EC2 actual de aplicaciones.

---

## Criterio rector

```text
USUARIO
   ↓
habla en conceptos de negocio

AGENTE
   ↓
selecciona capacidades

MCP TOOL
   ↓
expone contratos de negocio

CAPA DE DATOS / SERVICIO
   ↓
resuelve complejidad técnica

POSTGRESQL / SISTEMAS PROING
```

La arquitectura debe preservar esta separación a medida que el proyecto crezca.
