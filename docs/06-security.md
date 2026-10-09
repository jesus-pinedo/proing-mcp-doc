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

## Autenticación remota MVP

El endpoint productivo:

```text
https://mcp.proing.com.co/mcp
```

requiere autenticación por Bearer token individual por usuario.

Flujo:

```text
Cliente MCP
   ↓
HTTPS / Nginx
   ↓
Authorization: Bearer <token>
   ↓
validación Host / Origin
   ↓
TokenAuthenticator
   ↓
AuthContext
   ↓
handler MCP
```

Reglas:

- cada usuario recibe un token diferente;
- cada token se genera con 32 bytes aleatorios mediante CSPRNG;
- el servidor persiste únicamente `SHA-256(token)`;
- el token original se entrega una sola vez al usuario y se configura en el cliente MCP;
- una request sin token, con token inválido o perteneciente a un usuario deshabilitado recibe `401 Unauthorized`;
- todos los fallos de autenticación devuelven la misma respuesta observable;
- la autenticación ocurre antes de `toNodeHandler` y del handler MCP;
- `stdio` local permanece sin autenticación.

La respuesta HTTP uniforme es:

```text
401 Unauthorized
WWW-Authenticate: Bearer realm="proing-mcp"
Cache-Control: no-store
```

con body:

```json
{"error":"unauthorized"}
```

### Repositorio de tokens

La configuración productiva utiliza:

```text
MCP_AUTH_TOKENS_FILE=/etc/proing-mcp/tokens.json
```

Formato:

```json
{
  "version": 1,
  "users": [
    {
      "id": "usuario",
      "tokenHash": "<sha256 hexadecimal>",
      "enabled": true
    }
  ]
}
```

El archivo:

- vive fuera del repositorio;
- debe ser legible por el usuario del servicio y no por usuarios generales;
- se valida estrictamente al iniciar HTTP;
- se carga una sola vez antes de `listen()`;
- no se relee por request;
- requiere reinicio ordenado del servicio después de una rotación o revocación.

En producción se utiliza el usuario/grupo de servicio `proing-mcp` y permisos restrictivos sobre `/etc/proing-mcp`.

### Administración de tokens en producción

La administración se realiza con la CLI compilada:

```text
/opt/proing-mcp/app/dist/security/admin/auth-cli.js
```

y el binario Node:

```text
/usr/bin/node
```

Comandos operativos:

```bash
# Listar usuarios y estado
sudo env \
  MCP_AUTH_TOKENS_FILE=/etc/proing-mcp/tokens.json \
  /usr/bin/node \
  /opt/proing-mcp/app/dist/security/admin/auth-cli.js \
  list

# Crear usuario y generar token
sudo env \
  MCP_AUTH_TOKENS_FILE=/etc/proing-mcp/tokens.json \
  /usr/bin/node \
  /opt/proing-mcp/app/dist/security/admin/auth-cli.js \
  create <userId>

# Rotar token de un usuario existente
sudo env \
  MCP_AUTH_TOKENS_FILE=/etc/proing-mcp/tokens.json \
  /usr/bin/node \
  /opt/proing-mcp/app/dist/security/admin/auth-cli.js \
  rotate <userId>

# Deshabilitar usuario
sudo env \
  MCP_AUTH_TOKENS_FILE=/etc/proing-mcp/tokens.json \
  /usr/bin/node \
  /opt/proing-mcp/app/dist/security/admin/auth-cli.js \
  disable <userId>

# Habilitar usuario
sudo env \
  MCP_AUTH_TOKENS_FILE=/etc/proing-mcp/tokens.json \
  /usr/bin/node \
  /opt/proing-mcp/app/dist/security/admin/auth-cli.js \
  enable <userId>
```

Los `userId` deben cumplir:

```text
^[a-z][a-z0-9._-]{0,63}$
```

`create` y `rotate` muestran el token original una sola vez después de guardar correctamente el nuevo estado. Ese token debe entregarse por un canal seguro y configurarse en el cliente como:

```text
Authorization: Bearer <token>
```

El token no puede recuperarse después a partir del servidor; si se pierde debe rotarse.

Las mutaciones no reinician systemd. Después de `create`, `rotate`, `enable` o `disable` se debe ejecutar:

```bash
sudo systemctl restart proing-mcp
sudo systemctl is-active proing-mcp
```

Permisos productivos aprobados:

```text
/etc/proing-mcp                 root:proing-mcp 0750
/etc/proing-mcp/tokens.json     root:proing-mcp 0640
```

El usuario del servicio puede leer las credenciales pero no modificarlas. Las mutaciones se ejecutan mediante `sudo`; no se debe otorgar escritura del directorio al grupo `proing-mcp`.

La CLI protege la persistencia mediante archivo temporal, `fsync`, `rename` atómico y un lock local. Si un cierre abrupto deja un lock obsoleto, no debe eliminarse sin verificar antes que no exista otro proceso administrativo activo.

### Validación criptográfica

SHA-256 directo se utiliza únicamente porque los tokens tienen 256 bits de entropía y no son passwords elegidos por humanos.

El autenticador:

- limita la longitud del token antes de procesarlo;
- calcula SHA-256 sobre los bytes exactos;
- compara hashes mediante `timingSafeEqual`;
- rechaza IDs o hashes duplicados en configuración;
- no recorta ni normaliza silenciosamente el token.

### Limitaciones deliberadas del MVP

Esta fase no implementa todavía:

- OAuth;
- JWT propio;
- expiración automática de tokens;
- RBAC;
- permisos por Tool;
- data scopes;
- hot reload de credenciales.

Mientras este modelo esté vigente, todos los usuarios autenticados tienen acceso a las mismas Tools publicadas. No deben incorporarse Tools sensibles que requieran distintos niveles de autorización sin implementar antes la capa de autorización correspondiente.

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

No se registrarán secretos, el header `Authorization`, tokens ni hashes de tokens.

La autenticación actual ya permite identificar al actor mediante `userId` y `authType=static_token`. La auditoría estructurada por Tool y usuario se incorporará en la siguiente fase sin propagar secretos.

---

## Seguridad futura

El endpoint remoto ya dispone de HTTPS y autenticación Bearer individual para el MVP. La evolución de seguridad deberá cubrir:

- OAuth o identidad corporativa definitiva;
- autorización por usuario, rol, dominio y Tool;
- data scopes;
- auditoría estructurada por actor;
- rate limits;
- evolución futura de la administración de credenciales cuando el volumen lo requiera;
- revisión periódica de exposición de red y reverse proxy.

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
