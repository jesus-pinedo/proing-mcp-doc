# 09 — Definición Arquitectónica V1

**Estado:** Definición inicial  
**Versión:** 1.0  
**Fecha:** 29 de septiembre de 2026  
**Proyecto:** Proing MCP

---

## 1. Objetivo

Construir un servidor MCP corporativo para Proing que permita a agentes de inteligencia artificial acceder de forma controlada a capacidades y datos internos de la organización.

El servidor debe diseñarse desde el inicio para crecer hacia múltiples dominios, entre ellos:

- Operación
- Vehículos
- Inventario
- Personal
- Otros dominios futuros

La primera versión será deliberadamente pequeña y servirá para validar arquitectura, conectividad, protocolo, seguridad y experiencia de uso.

La V1 expondrá inicialmente una única capacidad relacionada con la consulta del histórico de vehículos.

---

## 2. Alcance de la V1

La primera versión comprenderá:

```text
Proing MCP
    │
    └── Operación
          │
          └── consultar histórico de vehículo
```

Nombre lógico inicialmente propuesto para la Tool:

```text
operacion.consultar_historico_vehiculo
```

El nombre definitivo y su contrato de entrada/salida se definirán en un documento funcional independiente.

La V1 tendrá:

```text
1 servidor MCP
1 dominio inicial: operación
1 Tool
0 Resources
0 Prompts corporativos
```

Resources y Prompts forman parte de la arquitectura futura, pero no se implementarán únicamente por estar disponibles en MCP.

---

## 3. Principios de diseño

### 3.1 El MCP se organiza por dominio

El servidor no se estructurará alrededor de tablas de base de datos.

Se estructurará alrededor de conceptos y capacidades de negocio.

Ejemplo futuro:

```text
PROING MCP

operacion.*
inventario.*
personal.*
mantenimiento.*
```

Internamente:

```text
domains/
    operacion/
    inventario/
    personal/
```

Esto permitirá agregar capacidades sin convertir el MCP en un conjunto desorganizado de consultas SQL.

### 3.2 El agente no conocerá el modelo físico de PostgreSQL

El agente no debe conocer:

- nombres de tablas;
- claves internas;
- relaciones;
- JOIN;
- estructuras históricas;
- nombres técnicos de columnas;
- SQL.

El agente utilizará conceptos de negocio.

Ejemplo:

```text
Usuario:

"Muéstrame el histórico del vehículo ABC123
entre el lunes y el miércoles."
```

El agente utilizará:

```text
operacion.consultar_historico_vehiculo
```

con parámetros estructurados.

No utilizará algo equivalente a:

```text
ejecutar_sql(...)
```

---

## 4. Arquitectura general V1

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

Para la primera versión no se creará obligatoriamente una API REST intermedia.

El MCP podrá consultar PostgreSQL directamente mediante un cliente PostgreSQL propio.

---

## 5. Estrategia de acceso a datos

Para capacidades de consulta y reporting se adopta inicialmente:

```text
MCP
 ↓
PostgreSQL
 ↓
Views / Functions controladas
 ↓
Tablas reales
```

Para futuras operaciones que modifiquen información o ejecuten procesos de negocio se favorecerá:

```text
MCP
 ↓
API / servicio de dominio
 ↓
reglas de negocio
 ↓
PostgreSQL
```

Por lo tanto, Proing MCP podrá utilizar diferentes mecanismos de integración dependiendo de la capacidad.

### Lectura / Reporting

Preferentemente:

```text
MCP → PostgreSQL
```

utilizando vistas, funciones o consultas controladas.

### Acciones de negocio

Preferentemente:

```text
MCP → API / Servicio
```

Ejemplos futuros:

```text
consultar productividad
        → PostgreSQL

consultar histórico
        → PostgreSQL

consultar consumo
        → PostgreSQL

registrar salida de inventario
        → API Inventario

cerrar orden
        → API Operación
```

---

## 6. Capa de datos para MCP

Se propone reservar un schema PostgreSQL para las superficies de datos destinadas al MCP.

Inicialmente:

```text
mcp
```

Ejemplo:

```text
PostgreSQL

operacion
├── tabla_a
├── tabla_b
├── tabla_c
└── ...

mcp
└── vw_historico_vehiculos
```

En dominios futuros podría existir:

```text
mcp.vw_historico_vehiculos
mcp.vw_ordenes
mcp.vw_productividad
mcp.fn_productividad(...)
mcp.fn_consumo_materiales(...)
```

El objetivo de esta capa es traducir:

```text
MODELO FÍSICO DE PROING
          ↓
MODELO ENTENDIBLE DE NEGOCIO
```

El usuario y el agente nunca deben necesitar comprender cómo están relacionadas las tablas originales.

---

## 7. Views versus Functions

Se utilizarán ambos mecanismos dependiendo del caso.

### View

Adecuada cuando existe una representación reutilizable y relativamente estable.

Ejemplo:

```text
mcp.vw_historico_vehiculos
```

El MCP podrá ejecutar consultas parametrizadas sobre esa vista.

Conceptualmente:

```text
vista
 +
placa
 +
fechaInicio
 +
fechaFin
```

### Function

Adecuada cuando la obtención del dato implique:

- múltiples JOIN;
- agregaciones;
- reglas;
- cálculos;
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

La complejidad permanece en PostgreSQL y el MCP consume un contrato de datos entendible.

---

## 8. Responsabilidad del MCP

El MCP será responsable de:

```text
MCP Tool
   ↓
validar entrada
   ↓
validar límites
   ↓
invocar capa de datos
   ↓
controlar errores
   ↓
normalizar resultado
   ↓
entregar respuesta estructurada
```

El MCP no debe asumir como responsabilidad principal:

```text
descubrir relaciones entre tablas
generar SQL arbitrario
interpretar esquemas físicos
permitir SQL enviado por el LLM
duplicar reglas existentes de procesos empresariales
```

---

## 9. Tecnología

La V1 utilizará:

```text
Runtime:
Node.js LTS

Lenguaje:
TypeScript

MCP:
MCP TypeScript SDK v2

Paquete servidor:
@modelcontextprotocol/server

Validación:
Zod

PostgreSQL:
pg

Gestor paquetes:
npm

Control de versiones:
Git

Repositorio:
GitLab Proing
```

Para el proyecto se utilizará una versión LTS vigente de Node.js y se mantendrá el proyecto en ESM.

---

## 10. Transportes MCP

El núcleo del servidor MCP no deberá depender de un transporte específico.

Conceptualmente:

```text
                  MCP CORE
                     │
             buildProingServer()
                     │
             ┌───────┴───────┐
             │               │
           stdio       Streamable HTTP
```

Esto permite utilizar exactamente las mismas Tools independientemente de cómo se conecte el cliente.

### 10.1 stdio

Será el mecanismo principal para desarrollo local.

```text
Agente local
    │
    │ inicia proceso
    ▼
node proing-mcp
    │
   stdio
```

Se utilizará para validar el MCP desde clientes locales compatibles.

### 10.2 Streamable HTTP

Además de `stdio`, la arquitectura permitirá iniciar el mismo MCP mediante:

```text
http://localhost:<puerto>/mcp
```

Este será el transporte esperado para el despliegue remoto posterior.

---

## 11. Compatibilidad con agentes

El MCP no deberá contener lógica específica para Claude, Gemini o ChatGPT.

Debe implementar correctamente el estándar MCP.

Conceptualmente:

```text
                 PROING MCP
                     ▲
           ┌─────────┼─────────┐
           │         │         │
         Claude    Gemini   ChatGPT
```

Cada host decidirá cómo presentar las Tools y cuándo utilizarlas.

Para desarrollo local se priorizarán clientes capaces de conectarse por `stdio` o Streamable HTTP.

Para clientes que requieran un servidor accesible remotamente, se utilizará posteriormente el transporte HTTP publicado de forma segura.

---

## 12. Estrategia de desarrollo local

La primera iteración deberá poder ejecutarse completamente desde una máquina de desarrollo.

```text
Mac / PC desarrollador
│
├── Node.js
│
├── Proing MCP
│
└── conexión PostgreSQL autorizada
```

No será necesario disponer inicialmente de la EC2 definitiva para desarrollar el servidor.

Una vez validado:

```text
local
 ↓
pruebas MCP
 ↓
prueba con agentes
 ↓
despliegue AWS
```

---

## 13. Acceso PostgreSQL

El MCP contará con un único pool de conexiones PostgreSQL compartido por los diferentes módulos.

Conceptualmente:

```text
Proing MCP

domains/
   │
   ├── operacion
   ├── inventario futuro
   └── ...
          │
          ▼
database/postgres
          │
          ▼
connection pool
          │
          ▼
PostgreSQL
```

No se abrirá una conexión nueva por cada Tool.

---

## 14. Usuario PostgreSQL dedicado

El MCP no utilizará las credenciales administrativas ni las credenciales generales de las aplicaciones existentes.

Se deberá crear un usuario dedicado, conceptualmente:

```text
proing_mcp
```

Su principio será:

```text
mínimo privilegio
```

Para la V1:

```text
SELECT únicamente
```

y acceso exclusivamente a las vistas/funciones necesarias.

Ejemplo:

```text
proing_mcp
     │
     └── SELECT
            ↓
mcp.vw_historico_vehiculos
```

No deberá tener permisos generales de escritura sobre la base de datos.

---

## 15. SQL controlado

No existirá una Tool del estilo:

```text
database.execute_sql
```

Ni se aceptará SQL generado por el agente.

Las consultas estarán controladas por el artefacto.

Ejemplo:

```text
operacion.consultar_historico_vehiculo
        ↓
repository / data access
        ↓
consulta parametrizada
        ↓
mcp.vw_historico_vehiculos
```

El agente proporciona únicamente filtros de negocio.

---

## 16. Estructura inicial del repositorio

Se propone:

```text
proing-mcp/
│
├── src/
│   │
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
│   │       │   └── consultar-historico-vehiculo.ts
│   │       │
│   │       └── repositories/
│   │           └── historico-vehiculo.repository.ts
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
│   │
│   ├── views/
│   │   └── vw_historico_vehiculos.sql
│   │
│   └── functions/
│
├── docs/
│   ├── architecture-v1.md
│   └── tools/
│       └── consultar-historico-vehiculo.md
│
├── tests/
│
├── .env.example
├── package.json
├── tsconfig.json
└── README.md
```

La estructura se mantendrá deliberadamente pequeña.

No se incorporarán patrones o capas adicionales sin una necesidad concreta.

---

## 17. Variables de configuración

La aplicación utilizará configuración externa al código.

Conceptualmente:

```text
MCP_NAME
MCP_VERSION

DATABASE_HOST
DATABASE_PORT
DATABASE_NAME
DATABASE_USER
DATABASE_PASSWORD

DATABASE_POOL_MAX

HTTP_PORT
```

`.env` podrá utilizarse exclusivamente en desarrollo local.

Las credenciales reales nunca deberán ser versionadas.

En AWS se evaluará posteriormente Secrets Manager u otro mecanismo corporativo para gestión de secretos.

---

## 18. Seguridad V1

La primera versión será exclusivamente de lectura.

Controles mínimos:

```text
Tool
 ↓
schema Zod
 ↓
validación de parámetros
 ↓
límites funcionales
 ↓
consulta parametrizada
 ↓
usuario PostgreSQL read-only
 ↓
vista / función autorizada
```

Cuando sea desplegado en AWS:

```text
EC2 MCP
    │
    │ red privada / Security Group
    ▼
PostgreSQL
```

PostgreSQL no deberá exponerse públicamente para permitir el acceso del MCP.

---

## 19. Límites de consulta

Cada Tool deberá definir explícitamente límites para evitar consultas descontroladas.

Dependiendo de la Tool podrán existir:

```text
fecha mínima/máxima
rango máximo de fechas
cantidad máxima de registros
paginación
timeout
filtros obligatorios
```

Estos límites pertenecen al contrato de cada Tool y no serán decididos libremente por el LLM.

---

## 20. Errores

La capa MCP deberá distinguir como mínimo entre:

```text
entrada inválida
sin resultados
error de base de datos
timeout
error interno
```

No deberá devolver al modelo:

- contraseñas;
- connection strings;
- stack traces internos;
- SQL sensible;
- detalles innecesarios del esquema físico.

---

## 21. Logging

Desde la primera versión se deberá registrar como mínimo:

```text
timestamp
tool
duración
resultado éxito/error
cantidad de registros
```

No deberán registrarse secretos.

Cuando exista autenticación de usuarios se incorporará identificación del actor de forma apropiada.

---

## 22. Resources y Prompts

La V1 no los necesita.

La arquitectura permitirá incorporarlos posteriormente.

Ejemplo futuro:

```text
TOOLS
operacion.consultar_historico_vehiculo
operacion.consultar_productividad
inventario.consultar_existencia

RESOURCES
proing://operacion/catalogos/estados
proing://inventario/bodegas

PROMPTS
analizar_operacion_vehiculo
generar_reporte_inventario
```

Los Prompts corporativos corresponderán principalmente a tareas estandarizadas de Proing.

Las plantillas personales y reportes frecuentes de cada usuario podrán pertenecer posteriormente al producto/agente cliente y no necesariamente al servidor MCP.

---

## 23. Evolución esperada

La arquitectura deberá permitir evolucionar sin modificar el principio básico.

### V1

```text
operacion
   └── consultar histórico vehículo
```

### Futuro

```text
PROING MCP

├── operacion
│   ├── consultar histórico
│   ├── consultar órdenes
│   └── consultar productividad
│
├── inventario
│   ├── consultar existencias
│   └── consultar consumos
│
└── otros dominios
```

Solamente se evaluará separar dominios en servidores MCP independientes cuando existan motivos reales como:

```text
seguridad independiente
equipos independientes
infraestructura independiente
ciclos de despliegue independientes
escala considerable
```

La cantidad de Tools por sí sola no obligará a crear múltiples servidores MCP.

---

## 24. Despliegue futuro

La V1 se desarrolla primero localmente.

Posteriormente:

```text
Internet / clientes autorizados
          │
          ▼
https://mcp.proing.com.co/mcp
          │
          ▼
      EC2 MCP
          │
          ▼
PostgreSQL / APIs Proing
```

El servidor MCP se desplegará en una instancia separada de la EC2 actual donde viven Apache y los procesos PM2 existentes.

La máquina MCP podrá convertirse posteriormente en la infraestructura destinada a capacidades de IA e integraciones de Proing.

---

## 25. Decisiones arquitectónicas V1

| Decisión | V1 |
|---|---|
| Servidor | Un único Proing MCP |
| Arquitectura | Modular por dominio |
| Primer dominio | Operación |
| Primera capacidad | Histórico de vehículos |
| Runtime | Node.js LTS |
| Lenguaje | TypeScript |
| MCP SDK | TypeScript SDK v2 |
| Validación | Zod |
| Base de datos | PostgreSQL |
| Driver PostgreSQL | `pg` |
| Acceso V1 | Directo desde MCP |
| Modelo datos | Views / Functions controladas |
| Usuario DB | Exclusivo y read-only |
| SQL generado por agente | No permitido |
| Desarrollo local | Sí |
| Transporte local | stdio |
| Transporte HTTP | Streamable HTTP |
| Resources V1 | No |
| Prompts V1 | No |
| API intermedia | No requerida para esta Tool |
| Repositorio de código | GitLab |
| Producción futura | EC2 separada |

---

## 26. Criterio rector

La arquitectura de Proing MCP deberá mantener siempre la siguiente separación:

```text
USUARIO
   ↓
habla en términos de negocio

AGENTE
   ↓
decide qué capacidad necesita

MCP TOOL
   ↓
expone una capacidad de negocio

CAPA DE DATOS / SERVICIO
   ↓
resuelve la complejidad técnica

POSTGRESQL / SISTEMA PROING
```

El usuario no deberá conocer la estructura física de los sistemas de Proing y el agente no deberá recibir acceso genérico a dicha estructura.

---

## 27. Próximo paso

El siguiente artefacto será el contrato funcional de:

```text
operacion.consultar_historico_vehiculo
```

Ese documento deberá definir, antes de implementar código:

```text
objetivo
casos de uso
parámetros de entrada
parámetros opcionales
límites
origen de datos
vista PostgreSQL
campos de salida
paginación
manejo de fechas
ordenamiento
errores
ejemplos de preguntas que debe resolver
criterios de aceptación
```

Solo después de cerrar ese contrato se iniciará la implementación de la primera Tool.
