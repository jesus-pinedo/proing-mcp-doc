# 09 — Plan de implementación MVP

**Proyecto:** Proing MCP  
**Versión:** V1 / MVP técnico  
**Estado:** Bloque 0 cerrado / Bloque 1 listo para ejecutar  
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

## Objetivo

Crear el proyecto mínimo ejecutable.

## Estructura inicial esperada

```text
proing-mcp/
│
├── src/
│   ├── server/
│   ├── transports/
│   ├── domains/
│   │   └── operacion/
│   │       ├── tools/
│   │       ├── repositories/
│   │       └── catalogs/
│   ├── infrastructure/
│   │   └── database/
│   ├── config/
│   └── index.ts
│
├── database/
│   └── views/
│
├── tests/
├── .env.example
├── .gitignore
├── package.json
└── tsconfig.json
```

## Dependencias esperadas

Como mínimo:

```text
@modelcontextprotocol/server
zod
pg
```

Más dependencias únicamente cuando sean necesarias para ejecutar o probar el proyecto.

## Validaciones

- TypeScript compila.
- El proyecto utiliza ESM.
- Existe un comando de desarrollo.
- Existe un comando de build.
- Existe un comando de test.
- `.env` está ignorado por Git.
- `.env.example` no contiene secretos.

## Criterio de cierre

Proyecto vacío pero compilable y ejecutable.

---

# 6. Bloque 2 — Configuración y conexión PostgreSQL

## Objetivo

Implementar configuración tipada y un único pool PostgreSQL reutilizable.

## Archivos conceptuales

```text
src/config/env.ts
src/infrastructure/database/postgres.ts
```

## Reglas

- no crear una conexión por Tool;
- utilizar un único pool;
- validar variables requeridas al iniciar;
- no imprimir password ni connection string;
- soportar cierre ordenado del pool;
- permitir timeout de consulta configurable.

## Prueba mínima

Ejecutar una consulta inocua de conectividad, por ejemplo:

```text
SELECT 1
```

No consultar todavía el histórico desde la Tool.

## Criterio de cierre

El proyecto puede abrir y cerrar correctamente una conexión PostgreSQL mediante el pool configurado.

---

# 7. Bloque 3 — Superficie PostgreSQL del histórico

## Objetivo

Crear una superficie controlada para lectura del histórico.

## Fuente

```text
transportes.tr_vehiculos_tso_historico
```

## Vista propuesta

```text
mcp.vw_historico_vehiculos
```

Campos lógicos:

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

## Reglas

- `id_interno` corresponde a `tso_id_`;
- no se devuelve luego al agente;
- `placa` corresponde a `tso_placa`;
- `fecha_hora` corresponde a `tso_fecha_hora`;
- `direccion` corresponde a `tso_direccion`;
- `velocidad` corresponde a `tso_velocidad`;
- `evento_valor_origen` corresponde a `tso_evento`;
- latitud y longitud deben quedar preparadas para consumo numérico seguro.

La View no debe contener el catálogo de eventos del MVP. Esa normalización vive en el MCP.

## SQL versionado

El script de creación debe quedar en:

```text
database/views/vw_historico_vehiculos.sql
```

## Validaciones

Probar al menos:

- registro con coordenadas válidas;
- coordenadas nulas;
- evento nulo;
- evento conocido;
- múltiples placas;
- rango de fechas.

## Criterio de cierre

La View devuelve únicamente la superficie necesaria para el MCP y no requiere acceso directo de la Tool a las columnas `tso_*`.

---

# 8. Bloque 4 — Catálogo JSON de eventos

## Objetivo

Implementar la normalización Proing del evento dentro del dominio de Operación.

## Archivo

```text
src/domains/operacion/catalogs/eventos-vehiculo.json
```

Cada entrada deberá contener:

```json
{
  "valor_origen": "Ignition ON",
  "codigo": "ENCENDIDO",
  "nombre": "Encendido",
  "descripcion": ""
}
```

El catálogo debe incluir todos los valores definidos en `docs/03-tools.md`.

## Comportamiento

Evento conocido:

```text
Ignition ON
→ ENCENDIDO
→ Encendido
```

Evento nuevo no catalogado:

```text
codigo = NO_CATALOGADO
nombre = Evento no catalogado
valor_origen = valor recibido
descripcion = ""
```

Evento nulo o vacío:

```text
codigo = SIN_EVENTO
nombre = Sin evento
valor_origen = null
descripcion = ""
```

## Reglas

- conservar siempre el valor original cuando exista;
- no fallar por evento desconocido;
- la búsqueda debe ser determinística;
- no modificar PostgreSQL para resolver este catálogo en el MVP.

## Tests mínimos

- evento conocido;
- `PANICO`;
- `PÁNICO`;
- `Panic button`;
- evento desconocido;
- null;
- string vacío.

## Criterio de cierre

El dominio puede transformar cualquier `tso_evento` válido, desconocido o vacío al contrato definido.

---

# 9. Bloque 5 — Repository y paginación

## Objetivo

Implementar acceso al histórico desde el MCP sin exponer SQL al agente.

## Archivo conceptual

```text
src/domains/operacion/repositories/historico-vehiculos.repository.ts
```

## Entrada interna

```text
placas[]
fecha_inicio
fecha_fin
limit
cursor?
```

## Consulta

Debe utilizar SQL parametrizado.

Filtro conceptual:

```text
placa IN (...)
fecha_hora >= fecha_inicio
fecha_hora <= fecha_fin
```

## Orden

```text
fecha_hora ASC
placa ASC
id_interno ASC
```

## Paginación

Implementar keyset pagination mediante cursor opaco.

El cursor puede representar internamente:

```text
fecha_hora
placa
id_interno
```

El cliente/agente no debe construir ni modificar estos componentes directamente.

Para saber si existe otra página:

```text
requested_limit = N
SQL LIMIT = N + 1
```

Si llegan `N + 1` registros:

```text
devolver N
has_more = true
next_cursor = cursor del último registro retornado
```

No ejecutar `COUNT(*)` para paginar.

## Seguridad

- SQL parametrizado;
- placas como valores, nunca interpoladas;
- cursor validado antes de utilizarse;
- límites aplicados en servidor.

## Criterio de cierre

Es posible recorrer un resultado de varias páginas sin duplicar ni omitir registros.

---

# 10. Bloque 6 — Tool MCP

## Objetivo

Registrar la Tool:

```text
operacion.consultar_historico_vehiculos
```

## Contrato

Seguir exactamente `docs/03-tools.md`.

Entrada:

```text
placas
fecha_inicio
fecha_fin
limit?
cursor?
```

Salida:

```text
consulta
paginacion
registros[]
```

Cada registro debe devolver:

```text
placa
fecha_hora
latitud
longitud
direccion
velocidad
evento {
  codigo
  nombre
  valor_origen
  descripcion
}
```

## Validaciones Zod

Como mínimo:

- `placas` no vacío;
- fechas válidas;
- `fecha_inicio <= fecha_fin`;
- máximo 31 días;
- `limit` dentro de valores permitidos;
- cursor válido cuando exista.

## Errores funcionales

Implementar los códigos documentados:

```text
INVALID_DATE_RANGE
INVALID_PLATES
DATA_SOURCE_ERROR
```

Sin resultados no es error.

## Criterio de cierre

La Tool puede invocarse programáticamente y devuelve exactamente el contrato documentado.

---

# 11. Bloque 7 — Servidor MCP y transporte stdio

## Objetivo

Crear el servidor MCP reutilizable y exponerlo localmente mediante `stdio`.

## Archivos conceptuales

```text
src/server/create-server.ts
src/transports/stdio.ts
```

## Regla

El registro de Tools debe ocurrir en el servidor/core, no dentro del transporte.

```text
createProingServer()
        │
        └── registra Tools

stdio.ts
        │
        └── conecta transporte
```

## Validaciones

- el cliente MCP puede descubrir la Tool;
- la descripción de la Tool es entendible;
- puede ejecutarse una consulta real;
- errores se devuelven de forma controlada;
- cerrar cliente no deja conexiones colgadas.

## Criterio de cierre

Un cliente MCP local puede listar e invocar `operacion.consultar_historico_vehiculos`.

---

# 12. Bloque 8 — Streamable HTTP

## Objetivo

Exponer el mismo servidor MCP mediante Streamable HTTP sin duplicar lógica.

## Archivo conceptual

```text
src/transports/http.ts
```

Endpoint local conceptual:

```text
http://localhost:<HTTP_PORT>/mcp
```

## Regla

No duplicar:

- Tool;
- schemas;
- repository;
- catálogo;
- reglas de negocio.

El transporte solo debe adaptar la conexión MCP.

## Seguridad MVP

El endpoint HTTP será únicamente para desarrollo/pruebas locales.

No implementar todavía autenticación productiva.

## Criterio de cierre

La misma Tool funciona tanto por `stdio` como por Streamable HTTP.

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
