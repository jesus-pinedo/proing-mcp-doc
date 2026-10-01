# 09 — Plan de implementación MVP

**Proyecto:** Proing MCP  
**Versión:** V1 / MVP técnico  
**Estado:** Bloques 0–8 cerrados / Bloque 9 listo para ejecutar  
**Primera Tool:** `operacion.consultar_historico_vehiculos`

---

## 1. Propósito

Este documento define la secuencia de implementación del primer MVP de Proing MCP.

La implementación se realizará **por bloques secuenciales**. Cada bloque debe tener un objetivo concreto, modificar únicamente lo necesario, incluir validaciones y quedar cerrado antes de avanzar al siguiente.

Codex no debe reinterpretar la arquitectura ni agregar librerías, capas, patrones o funcionalidades no solicitadas.

La fuente de verdad funcional y arquitectónica permanece en:

- `docs/02-architecture.md`
- `docs/03-tools.md`
- `docs/06-security.md`
- `docs/07-decisions.md`

---

# 2. Alcance del MVP

El MVP debe producir un servidor MCP local capaz de exponer:

```text
operacion.consultar_historico_vehiculos
```

La Tool deberá:

- recibir una o varias placas;
- recibir fecha inicial y fecha final;
- permitir un rango máximo de 31 días;
- consultar PostgreSQL directamente;
- devolver histórico paginado;
- devolver coordenadas, dirección, velocidad y evento;
- normalizar el evento mediante un catálogo JSON incluido en el MCP;
- funcionar inicialmente mediante `stdio`;
- disponer también de entrada Streamable HTTP;
- operar exclusivamente en modo lectura.

Fuera del alcance del MVP:

- autenticación remota definitiva;
- autorización por usuario;
- interfaz gráfica;
- Resources;
- Prompts;
- Tools adicionales;
- escritura sobre PostgreSQL;
- comparación optimizada de Excel masivos;
- despliegue productivo AWS.

---

# 3. Reglas que Codex no puede cambiar

Las siguientes decisiones están cerradas:

```text
Node.js LTS
TypeScript
ESM
MCP TypeScript SDK
Zod
pg
npm
PostgreSQL directo para lectura/reporting
arquitectura modular por dominio
usuario PostgreSQL read-only
sin SQL generado por LLM
catálogo de eventos en JSON
paginación por cursor
stdio local
Streamable HTTP adicional
```

No agregar:

- Prisma;
- ORM;
- NestJS;
- Docker;
- Redis;
- framework de arquitectura adicional;
- base de datos propia del MCP;
- API REST intermedia;
- sistema de autenticación;
- infraestructura AWS.

Si durante un bloque aparece una decisión no cubierta por la documentación, Codex debe detenerse y reportarla antes de asumir una solución.

---

# 4. Bloque 0 — Precondiciones

**Estado: CERRADO**

## Objetivo

Cerrar los datos de infraestructura y negocio indispensables antes de implementar.

### 4.1 Zona horaria

Se confirmó que `transportes.tr_vehiculos_tso_historico.tso_fecha_hora` es `timestamp without time zone` y se interpretará como:

```text
America/Bogota
UTC-05:00
```

Los registros recientes muestran que `tso_fecha_hora` y `tso_fecha_server` utilizan el mismo reloj local. La salida MCP deberá exponer fecha/hora con offset explícito.

### 4.2 Conectividad PostgreSQL

Se validó conectividad y permiso de lectura sobre la tabla histórica con el usuario de desarrollo actual.

Las credenciales reales se almacenarán en `.env`, que no se versionará. El repositorio incluirá `.env.example` sin secretos.

Cuando se cree el usuario definitivo `proing_mcp`, sus credenciales reemplazarán las temporales de desarrollo.

### 4.3 Límites iniciales

```text
DEFAULT_PAGE_SIZE = 1000
MAX_PAGE_SIZE = 5000
MAX_DATE_RANGE_DAYS = 31
APP_TIMEZONE = America/Bogota
```

El máximo de placas y el timeout se definirán después de las mediciones de rendimiento.

## Criterio de cierre

Cumplido. Puede iniciarse el Bloque 1.

---

# 5. Bloque 1 — Bootstrap Node.js + TypeScript

**Estado: CERRADO**

## Objetivo

Crear el proyecto mínimo ejecutable.

## Resultado

Se creó la estructura base documentada con:

- Node.js 24 LTS;
- TypeScript;
- ESM;
- npm;
- MCP TypeScript SDK;
- Zod;
- pg;
- scripts de desarrollo, build, start y test;
- `.env.example`;
- `.gitignore`;
- test mínimo de bootstrap.

Versiones reportadas:

```text
Node.js 24.21.0
npm 11.19.0
@modelcontextprotocol/server 2.2.0
zod 4.6.5
pg 8.23.0
typescript 7.0.2
tsx 4.23.15
@types/node 26.6.3
```

Validaciones:

- `npm install`: exitoso;
- `npm run build`: exitoso;
- `npm start`: exitoso;
- `npm test`: 1 aprobado, 0 fallidos;
- sin secretos detectados;
- no se implementaron elementos de bloques posteriores.

## Criterio de cierre

Cumplido.

---

# 6. Bloque 2 — Configuración y conexión PostgreSQL

**Estado: CERRADO**

## Objetivo

Implementar configuración tipada y un único pool PostgreSQL reutilizable.

## Resultado

Se implementó:

- `src/config/env.ts`;
- `src/infrastructure/database/postgres.ts`;
- `src/infrastructure/database/check-connection.ts`;
- validación tipada de variables;
- carga nativa de `.env` mediante `process.loadEnvFile`;
- pool PostgreSQL único y reutilizable;
- soporte opcional de `query_timeout`;
- cierre ordenado del pool;
- script `npm run db:check`;
- tests unitarios de configuración y singleton del pool.

Se agregó `@types/pg@8.23.1` como dependencia de desarrollo.

Validaciones finales:

```text
npm run db:check  → exitoso
npm run build     → exitoso
npm test          → 7 aprobados / 0 fallidos
```

No se implementaron consultas históricas, Views, repositories, catálogo, Tools ni transportes.

## Criterio de cierre

Cumplido.

---

# 7. Bloque 3 — Superficie PostgreSQL del histórico

**Estado: CERRADO**

## Resultado

Se implementó y ejecutó:

```text
mcp.vw_historico_vehiculos
```

Campos expuestos:

```text
id_interno
placa
fecha_hora
latitud
longitud
direccion
velocidad
evento_valor_origen
```

La View:

- no expone `tso_fecha_server`;
- conserva `fecha_hora` sin conversión;
- conserva el evento original;
- convierte latitud/longitud de forma segura;
- devuelve NULL para coordenadas inválidas o fuera de rango;
- no incorpora catálogo de eventos;
- no agrega índices ni permisos.

Mediciones relevantes:

```text
1 placa × 1 día: ~1.36 ms
1 placa × 30 días: ~1.18 s en lectura fría
1 placa × 30 días: ~12 ms con datos en caché
```

El índice existente `(tso_placa, tso_fecha_hora)` fue utilizado correctamente. No se requieren índices nuevos para el MVP.

La clave lógica para la futura paginación queda simplificada a:

```text
fecha_hora
placa
```

aprovechando la unicidad de `(placa, fecha_hora)`.

## Criterio de cierre

Cumplido.

---

# 8. Bloque 4 — Catálogo JSON de eventos

**Estado: CERRADO**

## Resultado

Se implementó:

```text
src/domains/operacion/catalogs/eventos-vehiculo.json
src/domains/operacion/catalogs/vehicle-events.catalog.ts
tests/vehicle-events.catalog.test.ts
```

El catálogo contiene 27 eventos y la normalización:

- conserva `valor_origen`;
- utiliza lookup exacto y determinístico;
- normaliza variantes de pánico;
- devuelve `NO_CATALOGADO` para valores desconocidos;
- devuelve `SIN_EVENTO` para null/vacío/espacios;
- mantiene `descripcion` vacía en V1.

Se habilitó `resolveJsonModule` para importar el catálogo JSON.

Validaciones:

```text
npm run build → exitoso
npm test      → 18 aprobados / 0 fallidos
```

No se modificaron PostgreSQL, View, repository, cursor, Tool ni transportes.

## Criterio de cierre

Cumplido.

---

# 9. Bloque 5 — Repository y paginación

**Estado: CERRADO**

## Resultado

Se implementó:

```text
src/domains/operacion/repositories/vehicle-history.repository.ts
src/domains/operacion/repositories/vehicle-history.cursor.ts
tests/vehicle-history.repository.test.ts
tests/vehicle-history.cursor.test.ts
```

Características:

- consulta exclusivamente `mcp.vw_historico_vehiculos`;
- SQL parametrizado;
- una o varias placas;
- keyset pagination;
- cursor base64url versionado;
- clave lógica `fecha_hora + placa`;
- orden `fecha_hora ASC, placa ASC`;
- `LIMIT N + 1`;
- sin `COUNT(*)`;
- sin `OFFSET`;
- fechas leídas como texto local explícito con seis dígitos de microsegundos;
- sin dependencia de zona horaria de Node.js;
- evento conservado como `eventoValorOrigen`;
- testabilidad mediante ejecutor mínimo inyectable.

Validaciones:

```text
npm run build → exitoso
npm test      → 37 aprobados / 0 fallidos
```

## Criterio de cierre

Cumplido.

---

# 10. Bloque 6 — Tool MCP

**Estado: CERRADO**

## Resultado

Se implementó:

```text
src/domains/operacion/tools/vehicle-history.tool.ts
tests/vehicle-history.tool.test.ts
```

La Tool:

- expone `operacion.consultar_historico_vehiculos`;
- valida entrada con Zod;
- valida salida con Zod;
- soporta una o varias placas;
- aplica límite de 31 días;
- usa page size 1000 por defecto y 5000 máximo;
- acepta cursor opcional;
- normaliza fechas de entrada hacia `America/Bogota`;
- conserva microsegundos y añade `-05:00` en salida;
- utiliza únicamente el repository existente;
- aplica el catálogo de eventos existente;
- no contiene SQL ni acceso directo a PostgreSQL.

Errores controlados:

```text
INVALID_PLATES
INVALID_DATE_RANGE
INVALID_CURSOR
DATA_SOURCE_ERROR
```

Validaciones:

```text
npm run build → exitoso
npm test      → 60 aprobados / 0 fallidos
```

No se implementaron servidor ni transportes.

## Criterio de cierre

Cumplido.

---

# 11. Bloque 7 — Servidor MCP y transporte stdio

**Estado: CERRADO**

## Resultado

Se implementó y validó:

- servidor MCP reusable con `McpServer`;
- registro único de `operacion.consultar_historico_vehiculos`;
- transporte `StdioServerTransport`;
- protección estricta de stdout;
- cierre idempotente de servidor y pool;
- manejo de SIGINT/SIGTERM;
- `tools/list` validado mediante MCP Inspector;
- invocación real exitosa contra PostgreSQL;
- respuesta real con eventos normalizados, coordenadas y fechas `-05:00`;
- manejo de error `INVALID_CURSOR` mediante `isError: true`;
- errores sin `structuredContent`, evitando validación contra el `outputSchema` exitoso.

Validaciones finales:

```text
npm run build    → exitoso
npm test         → 70 aprobados / 0 fallidos
npm run db:check → exitoso
tools/list       → exitoso
tools/call       → exitoso
```

El MCP local por stdio queda funcional de punta a punta.

## Criterio de cierre

Cumplido.

---

# 11.5. Bloque 7.5 — Refactor de responsabilidades

**Estado: CERRADO**

## Resultado

Se implementó la separación aprobada:

```text
Tool
 ↓
Contract
 ↓
Service
 ↓
Repository
```

Archivos principales:

```text
src/domains/operacion/tools/vehicle-history.tool.ts
src/domains/operacion/contracts/vehicle-history.contract.ts
src/domains/operacion/services/vehicle-history.service.ts
src/domains/operacion/repositories/vehicle-history.repository.ts
src/domains/operacion/repositories/vehicle-history.cursor.ts
src/domains/operacion/catalogs/vehicle-events.catalog.ts
```

Cambios:

- schemas Zod y tipos públicos movidos a Contract;
- lógica de fechas, catálogo y composición de respuesta movida a Service;
- Tool reducida a responsabilidades MCP;
- Repository sin cambios funcionales;
- `AGENTS.md` actualizado con SRP y reglas permanentes de capas;
- no se creó Controller;
- no se agregaron helpers/utils genéricos;
- no se añadieron dependencias.

Validaciones:

```text
npm run build    → exitoso
npm test         → 72 aprobados / 0 fallidos
npm run db:check → exitoso
tools/list       → exitoso
tools/call real  → exitoso
```

El comportamiento observable y el contrato MCP permanecen sin cambios.

## Nota sobre repositorios

El repositorio de código y el repositorio documental continúan separados. El Bloque 7.5 está definido en `proing-mcp-doc`; no es requisito duplicar el plan completo dentro del repositorio local de código.

## Criterio de cierre

Cumplido.

---

# 12. Bloque 8 — Streamable HTTP

**Estado: CERRADO**

## Resultado

Se implementó:

```text
src/transports/http.ts
src/transports/lifecycle.ts
tests/http.test.ts
```

Se agregó:

```text
@modelcontextprotocol/node@2.1.0
```

Características:

- `createMcpHandler(factory, { legacy: "stateless" })`;
- `toNodeHandler`;
- servidor `node:http`;
- escucha en `127.0.0.1`;
- endpoint `/mcp`;
- rutas distintas devuelven 404;
- validación local de Host y Origin;
- factory reutiliza `createProingServer()`;
- nuevo `McpServer` por request;
- pool PostgreSQL singleton compartido;
- sin sesiones, Redis, event store ni resumabilidad;
- lifecycle compartido y específico, sin helpers genéricos;
- `stdio` permanece funcional.

Validaciones:

```text
npm run build    → exitoso
npm test         → 78 aprobados / 0 fallidos
npm run db:check → exitoso
tools/list HTTP  → exitoso
tools/call HTTP  → exitoso
```

Una invocación real por HTTP devolvió registros con:

- `structuredContent`;
- fechas `-05:00`;
- coordenadas numéricas;
- evento normalizado;
- paginación correcta.

## Criterio de cierre

Cumplido.

---

# 13. Bloque 9 — Tests

## Objetivo

Cubrir el comportamiento crítico del MVP.

## Tests mínimos

### Validación

- placas vacías;
- fechas inválidas;
- rango mayor a 31 días;
- limit inválido.

### Catálogo

- evento conocido;
- evento equivalente;
- evento desconocido;
- evento nulo.

### Repository

- una placa;
- varias placas;
- sin resultados;
- primera página;
- página siguiente;
- última página;
- orden estable.

### Tool

- respuesta completa válida;
- error controlado de DB;
- resultado vacío.

### Transportes

Al menos una prueba/integración que demuestre que el servidor registra la Tool correctamente.

## Criterio de cierre

Todos los tests automatizados pasan y no requieren credenciales productivas embebidas.

---

# 14. Bloque 10 — Validación con cliente/agente

## Objetivo

Validar que el MCP pueda utilizarse desde un cliente real.

## Casos mínimos

### Caso 1

```text
Consulta el histórico de ABC123 del día X.
```

### Caso 2

```text
Consulta ABC123 y XYZ789 entre fecha A y fecha B.
```

### Caso 3

Consulta que requiera más de una página.

### Caso 4

Pregunta basada en eventos:

```text
¿Qué eventos de encendido y apagado tuvo el vehículo?
```

### Caso 5

Uso de coordenadas para análisis posterior por parte del agente.

## Validar

- el agente selecciona la Tool adecuada;
- construye correctamente los parámetros;
- sigue `next_cursor` cuando lo necesita;
- entiende los eventos normalizados;
- no necesita conocer nombres de tablas ni columnas físicas.

## Criterio de cierre

Un agente MCP compatible puede utilizar la Tool para responder una consulta real de histórico.

---

# 15. Bloque 11 — Medición de rendimiento

## Objetivo

Medir antes de modificar índices o arquitectura.

Ejecutar escenarios reales con `EXPLAIN ANALYZE`:

```text
1 placa  × 1 día
1 placa  × 30 días
5 placas × 30 días
10 placas × 30 días
```

Registrar:

- tiempo de ejecución;
- plan utilizado;
- cantidad aproximada de filas;
- uso del índice existente;
- comportamiento de páginas posteriores.

Índice existente relevante:

```text
UNIQUE (tso_placa, tso_fecha_hora)
```

No crear índices adicionales hasta medir.

## Criterio de cierre

Se documenta si el índice actual es suficiente para el MVP o si se requiere optimización.

---

# 16. Bloque 12 — Cierre documental

## Objetivo

Actualizar la documentación con la implementación real.

Actualizar como mínimo:

```text
docs/03-tools.md
docs/07-decisions.md
docs/08-current-status.md
```

Registrar:

- valores finales de límites;
- zona horaria adoptada;
- estrategia final de View;
- resultados de pruebas de rendimiento;
- cliente MCP utilizado para validar;
- pendientes detectados durante el MVP.

## Criterio de cierre

La documentación representa exactamente lo que existe en código.

---

# 17. Orden obligatorio de implementación

```text
Bloque 0  Precondiciones
   ↓
Bloque 1  Bootstrap
   ↓
Bloque 2  PostgreSQL
   ↓
Bloque 3  View
   ↓
Bloque 4  Catálogo JSON
   ↓
Bloque 5  Repository / cursor
   ↓
Bloque 6  Tool
   ↓
Bloque 7  stdio
   ↓
Bloque 8  Streamable HTTP
   ↓
Bloque 9  Tests
   ↓
Bloque 10 Agente
   ↓
Bloque 11 Rendimiento
   ↓
Bloque 12 Documentación final
```

No implementar bloques posteriores anticipadamente salvo que una dependencia técnica mínima lo requiera y quede documentada.

---

# 18. Resultado esperado del MVP

Al finalizar:

```text
Agente MCP
    ↓
operacion.consultar_historico_vehiculos
    ↓
Proing MCP
    ├── validación
    ├── catálogo JSON
    ├── cursor
    └── repository
          ↓
mcp.vw_historico_vehiculos
          ↓
transportes.tr_vehiculos_tso_historico
```

El resultado debe ser un MCP pequeño, entendible, probado y preparado para incorporar futuros dominios sin convertir el agente en conocedor de la estructura interna de Proing.
