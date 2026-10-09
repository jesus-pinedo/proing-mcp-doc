# 06 — Seguridad

## Objetivo

Definir desde el inicio límites claros para el acceso de agentes de IA a información interna de Proing.

La V1 será exclusivamente de lectura.

---

## Principios

### Menor privilegio

El MCP utilizará un usuario PostgreSQL dedicado, conceptualmente:

```text
proing_mcp
```

No utilizará credenciales administrativas ni usuarios generales de otras aplicaciones.

Para la V1 deberá tener únicamente los permisos necesarios para consultar las superficies de datos autorizadas.

Ejemplo:

```text
proing_mcp
     │
     └── SELECT / EXECUTE autorizado
              ↓
        schema mcp
              ↓
views / functions específicas
```

No deberá tener permisos generales de escritura sobre la base de datos.

### Sin SQL libre

El agente no debe enviar SQL arbitrario para ser ejecutado por el MCP.

No existirá una Tool del estilo:

```text
database.execute_sql
```

Las consultas deberán ser:

- parametrizadas;
- controladas por el servidor;
- realizadas sobre Views/Functions autorizadas o consultas definidas en la capa de datos.

### Contratos explícitos

Cada Tool debe definir:

- parámetros permitidos;
- tipos;
- validaciones;
- límites;
- errores controlados;
- alcance de datos.

El LLM no decidirá libremente los límites de consulta.

---

## Capas de protección V1

```text
Tool
 ↓
validación Zod
 ↓
límites funcionales
 ↓
consulta parametrizada
 ↓
usuario PostgreSQL dedicado
 ↓
View / Function autorizada
 ↓
datos permitidos
```

Cuando se despliegue en AWS:

```text
EC2 MCP
   │
   │ red privada / Security Group
   ▼
PostgreSQL
```

La base de datos no deberá exponerse públicamente para permitir el acceso del MCP.

---

## Límites de consulta

Cada Tool deberá establecer los límites que correspondan:

- filtros obligatorios;
- rango máximo de fechas;
- cantidad máxima de registros;
- paginación;
- timeout;
- tamaño máximo de respuesta.

Estos límites deberán quedar documentados en el contrato de cada Tool.

---

## Secretos fuera del repositorio

Las credenciales deben estar en variables de entorno o un sistema de secretos.

No deben versionarse:

- passwords;
- tokens;
- certificados privados;
- connection strings con credenciales.

En desarrollo local podrá utilizarse:

```text
.env
DATABASE_HOST=
DATABASE_PORT=
DATABASE_NAME=
DATABASE_USER=
DATABASE_PASSWORD=
```

El archivo `.env` no debe almacenarse en Git.

En AWS se evaluará Secrets Manager u otro mecanismo corporativo.

## Validación del transporte HTTP

Streamable HTTP mantiene protección explícita de Host y Origin mediante las
utilidades oficiales de `@modelcontextprotocol/node`.

En local, si no se configura lo contrario, las listas permitidas contienen
únicamente:

```text
localhost
127.0.0.1
[::1]
```

En producción, los hostnames públicos deben declararse explícitamente, sin
esquema, puerto ni comodines:

```env
HTTP_ALLOWED_HOSTS=localhost,127.0.0.1,mcp.proing.com.co
HTTP_ALLOWED_ORIGINS=localhost,127.0.0.1,mcp.proing.com.co
```

Un Host no autorizado se rechaza. Para Origin se conserva el comportamiento
del SDK: los clientes MCP no-browser pueden omitir el header; si lo envían, el
hostname debe estar autorizado. Esta validación no introduce CORS genérico.

El proceso Node escucha exclusivamente en `127.0.0.1` y permanece detrás de
Nginx, que expone el endpoint HTTPS productivo.

---

## Manejo de errores

El MCP deberá diferenciar al menos:

- entrada inválida;
- sin resultados;
- timeout;
- error de base de datos;
- error interno.

No deberá devolver al agente:

- contraseñas;
- connection strings;
- stack traces internos;
- SQL sensible;
- detalles innecesarios del esquema físico.

---

## Logging y auditoría

Desde la V1 se registrará como mínimo:

- timestamp;
- Tool;
- duración;
- éxito/error;
- cantidad de registros.

No se registrarán secretos.

Cuando exista autenticación de usuarios se incorporará identificación del actor y trazabilidad por usuario.

---

## Seguridad futura

Antes de publicar el MCP remotamente se deberá definir:

- autenticación;
- autorización;
- TLS;
- exposición de red;
- auditoría;
- rate limits;
- trazabilidad por usuario;
- alcance por dominio o Tool.

---

## Protección de datos

Que un dato exista en PostgreSQL no significa que deba exponerse al agente.

Cada nueva Tool deberá pasar por una revisión explícita de:

- datos expuestos;
- columnas necesarias;
- permisos;
- filtros;
- sensibilidad;
- alcance por usuario.
