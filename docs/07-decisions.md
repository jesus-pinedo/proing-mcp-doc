# 07 — Decisiones

Registro de decisiones arquitectónicas aprobadas para Proing MCP.

---

## DEC-001 — Node.js + TypeScript

**Estado:** Aprobada

El servidor MCP inicial se desarrollará con Node.js LTS y TypeScript.

Se utilizará el SDK oficial de MCP para TypeScript, validación con Zod y módulos ESM.

### Motivo

- ecosistema natural para MCP;
- tipado;
- facilidad de evolución;
- familiaridad con el stack;
- despliegue liviano.

---

## DEC-002 — PostgreSQL directo desde el MCP para lectura/reporting

**Estado:** Aprobada

La V1 se conectará directamente a PostgreSQL mediante un cliente `pg` y un pool compartido.

No se creará inicialmente una capa HTTP intermedia exclusivamente para envolver consultas de lectura.

### Motivo

Crear endpoints adicionales para consultas que únicamente consume el MCP introduce componentes y mantenimiento sin aportar suficiente valor al MVP.

---

## DEC-003 — API/servicio para acciones de negocio

**Estado:** Aprobada

Las operaciones que registren, modifiquen información o ejecuten procesos de negocio deberán preferentemente utilizar la API o servicio del dominio correspondiente.

### Motivo

Las reglas, transacciones y validaciones empresariales no deben duplicarse dentro del MCP.

---

## DEC-004 — La lógica compleja de datos no vive en el agente

**Estado:** Aprobada

Cuando una consulta requiera múltiples tablas, JOIN, agregaciones o reglas, esa complejidad deberá resolverse antes de entregar la información al agente.

Se priorizarán:

- Views PostgreSQL;
- Functions PostgreSQL;
- consultas controladas en la capa de datos.

El agente trabajará con capacidades y filtros de negocio.

---

## DEC-005 — Schema de exposición para MCP

**Estado:** Aprobada

Se propone utilizar un schema dedicado, conceptualmente `mcp`, para Views y Functions preparadas para consumo del MCP.

### Motivo

Permite separar el modelo físico de Proing de la superficie de datos autorizada para agentes.

---

## DEC-006 — Arquitectura por dominios

**Estado:** Aprobada

El MCP crecerá organizado por dominios.

Ejemplos:

- operación;
- inventario;
- personal.

Los Tools utilizarán nombres orientados a capacidades, por ejemplo:

`operacion.consultar_historico_vehiculos`

`inventario.consultar_existencia`

---

## DEC-007 — Un único MCP modular inicialmente

**Estado:** Aprobada

La V1 utilizará un solo servidor Proing MCP.

No se crearán servidores separados por dominio únicamente por aumentar la cantidad de Tools.

Se reevaluará cuando existan fronteras reales de seguridad, equipo, infraestructura, despliegue o escala.

---

## DEC-008 — Primer MVP local

**Estado:** Aprobada

El primer MCP se ejecutará localmente y deberá poder ser consumido por un cliente MCP compatible.

El núcleo será independiente del transporte.

---

## DEC-009 — stdio local y Streamable HTTP remoto

**Estado:** Aprobada

Durante el desarrollo local se utilizará principalmente `stdio`.

La arquitectura incluirá Streamable HTTP para pruebas y futuro despliegue remoto.

Las Tools no dependerán del transporte.

---

## DEC-010 — Primera Tool de histórico de vehículo

**Estado:** Aprobada

La primera capacidad será:

`operacion.consultar_historico_vehiculos`

Su contrato funcional se definirá antes de implementar.

---

## DEC-011 — Resources y Prompts no forman parte de la V1

**Estado:** Aprobada

La arquitectura contempla Tools, Resources y Prompts, pero la primera versión implementará únicamente la Tool necesaria.

Resources y Prompts se incorporarán cuando exista una necesidad concreta.

---

## DEC-012 — Usuario PostgreSQL dedicado y read-only

**Estado:** Aprobada

El MCP utilizará un usuario de base de datos exclusivo con principio de mínimo privilegio.

La V1 será exclusivamente de lectura.

---

## DEC-013 — Sin SQL generado por el agente

**Estado:** Aprobada

No se expondrá una capacidad genérica para ejecutar SQL suministrado por el LLM.

El agente únicamente suministrará filtros de negocio definidos por cada contrato.

---

## DEC-014 — Infraestructura productiva separada

**Estado:** Aprobada

Cuando se despliegue en AWS, el Proing MCP se ejecutará en una instancia separada de la EC2 actual de aplicaciones.

### Motivo

Evitar acoplar el nuevo artefacto a una instancia que ya ejecuta Apache y otros procesos, y reservar una frontera clara para futuras capacidades de IA/integración.

---

## DEC-015 — Documentación separada temporalmente

**Estado:** Aprobada

Durante la etapa inicial, la documentación se mantiene en este repositorio independiente.

El código se trabajará localmente y posteriormente se versionará en GitLab Proing.

La documentación podrá migrarse al repositorio definitivo cuando se decida consolidar ambos.


---

## DEC-016 — Catálogo de eventos en JSON para el MVP

**Estado:** Aprobada

El catálogo que normaliza los valores de `tso_evento` se mantendrá inicialmente como un archivo JSON versionado dentro del dominio de Operación del MCP.

Ubicación conceptual:

```text
src/domains/operacion/catalogs/eventos-vehiculo.json
```

PostgreSQL conservará y entregará el valor original del proveedor. La capa de dominio del MCP traducirá ese valor a código, nombre y descripción Proing.

### Motivo

- catálogo pequeño y estable;
- simplicidad para el MVP;
- versionado en Git;
- evita crear una tabla adicional únicamente para este artefacto;
- permite migrarlo posteriormente a una fuente dinámica sin cambiar el contrato de la Tool.


---

## DEC-017 — Zona horaria del histórico

**Estado:** Aprobada

La columna `transportes.tr_vehiculos_tso_historico.tso_fecha_hora`, definida como `timestamp without time zone`, se interpretará en la V1 como hora local de Colombia:

```text
America/Bogota
UTC-05:00
```

### Evidencia

En registros recientes, `tso_fecha_hora` y `tso_fecha_server` utilizan el mismo reloj local y normalmente presentan diferencias de segundos o pocos minutos, sin un desfase sistemático de cinco horas.

Se observó al menos un registro con hora del dispositivo adelantada respecto a la hora del servidor. Este comportamiento se considera una posible anomalía del dispositivo/proveedor, no un cambio de zona horaria.

### Regla de contrato

El MCP interpretará `tso_fecha_hora` como `America/Bogota` y expondrá fechas con offset explícito.

---

## DEC-018 — Credenciales locales mediante .env

**Estado:** Aprobada

Las credenciales reales de PostgreSQL para desarrollo local se configurarán en un archivo `.env` no versionado.

El repositorio incluirá únicamente `.env.example` sin secretos.

El usuario definitivo `proing_mcp` podrá crearse posteriormente; al estar disponible, sus credenciales sustituirán las credenciales temporales de desarrollo en el `.env`.

Variables mínimas:

```text
DATABASE_HOST
DATABASE_PORT
DATABASE_NAME
DATABASE_USER
DATABASE_PASSWORD
APP_TIMEZONE=America/Bogota
```


---

## DEC-019 — Cursor basado en fecha_hora + placa

**Estado:** Aprobada

La paginación de `operacion.consultar_historico_vehiculos` utilizará como clave lógica:

```text
fecha_hora
placa
```

No se incluirá `id_interno` en el cursor.

### Motivo

La tabla fuente posee una restricción única sobre:

```text
(tso_placa, tso_fecha_hora)
```

Por lo tanto, la combinación lógica `(fecha_hora, placa)` identifica de forma determinística cada registro consultable y permite simplificar el cursor.

---

## DEC-020 — No crear índice adicional para el MVP

**Estado:** Aprobada

No se crearán índices adicionales para la primera versión.

### Evidencia

El índice existente:

```text
UNIQUE (tso_placa, tso_fecha_hora)
```

fue utilizado correctamente por PostgreSQL en las pruebas.

Las comprobaciones preliminares confirmaron que PostgreSQL podía utilizar el
índice. La evidencia definitiva para decidir sobre índices es la medición
controlada del Bloque 11 que se registra a continuación.

### Validación final de rendimiento — Bloque 11

Las consultas reales del Repository se midieron con
`EXPLAIN (ANALYZE, BUFFERS)`, `DEFAULT_PAGE_SIZE=100` y `LIMIT 101`:

| Escenario | Trabajo observado | Execution Time | Shared read |
|---|---|---:|---:|
| 1 placa × 1 día | 66 filas, Index Scan | 0,95–1,91 ms | 0 |
| 1 placa × 30 días | 1.402 candidatos, Bitmap Index/Heap Scan | 6,19–13,32 ms | 0 |
| 5 placas × 30 días | 33.398 entradas examinadas, 33.296 descartadas | 31,39–41,43 ms | 0 |
| 10 placas × 30 días | 6.312 entradas examinadas, 6.210 descartadas | 5,70–10,62 ms | 0 |

El escenario de cinco placas mostró trabajo adicional por el filtro de placas,
pero no un cuello de botella crítico. El escenario de diez placas fue más
rápido porque encontró coincidencias con mayor densidad en el índice temporal;
por tanto, el costo de la primera página depende también de la distribución de
los datos y no crece linealmente con la cantidad de placas.

Las páginas posteriores de `TUZ64G` para septiembre confirmaron la efectividad
del cursor keyset:

```text
primera página:       ~6,56–6,73 ms / 1.361 buffers hit
página intermedia:    ~3,36–3,37 ms / 686 buffers hit
página final:         ~0,13–0,15 ms / 6 buffers hit
```

El cursor excluye las filas ya entregadas mediante el `Index Cond`, sin la
degradación acumulativa de `OFFSET`. No se observaron lecturas físicas en estas
mediciones.

### Regla

Los índices actuales y el cursor keyset son suficientes para el MVP. No se
agregará un índice nuevo hasta que el uso real o futuras mediciones demuestren
una necesidad concreta y repetible.


---

## DEC-021 — Fecha histórica como texto local en la capa de datos

**Estado:** Aprobada

El repository de histórico obtiene `fecha_hora` desde PostgreSQL como texto local explícito con microsegundos:

```text
YYYY-MM-DDTHH:mm:ss.ffffff
```

No se crea un objeto JavaScript `Date` a partir del `timestamp without time zone` de PostgreSQL.

### Motivo

Evitar conversiones implícitas de zona horaria dependientes de Node.js, sistema operativo o configuración del proceso.

La capa pública de la Tool será responsable de añadir el offset aprobado de Colombia (`-05:00`) al serializar la respuesta MCP.


---

## DEC-022 — INVALID_CURSOR como error funcional

**Estado:** Aprobada

La Tool `operacion.consultar_historico_vehiculos` incorpora el código de error:

```text
INVALID_CURSOR
```

para cursores corruptos, estructuralmente inválidos o no soportados.

El error debe ser controlado y no exponer detalles internos de codificación.

---

## DEC-023 — Schema estricto de salida para la Tool

**Estado:** Aprobada

La primera Tool define y valida también su contrato público de salida mediante Zod.

### Motivo

- evitar fugas accidentales de campos internos;
- mantener estable el contrato MCP;
- detectar desviaciones entre repository, transformación de dominio y respuesta pública;
- facilitar pruebas automatizadas del contrato.

Esto no implica crear una capa genérica de schemas para futuras Tools; cada Tool podrá definir únicamente lo necesario.


---

## DEC-024 — Separación Tool / Contract / Service / Repository

**Estado:** Aprobada

Para Tools con suficiente complejidad, Proing MCP utilizará la siguiente separación de responsabilidades:

```text
Tool
 ↓
Contract
 ↓
Service
 ↓
Repository
```

### Responsabilidades

- **Tool**: adaptación al protocolo MCP, metadata, registro, invocación y traducción de errores.
- **Contract**: schemas Zod públicos de entrada/salida y tipos derivados.
- **Service**: orquestación del caso de uso, normalización de fechas, composición de respuesta y uso de catálogos.
- **Repository**: persistencia, SQL, paginación y acceso a fuentes de datos.

### Regla

No se crearán Controllers adicionales para las Tools MCP: la Tool ya representa la frontera equivalente al controller/adaptador.

No todas las Tools requieren Service; se introduce cuando existe lógica de aplicación suficiente para separar responsabilidades.

No se utilizarán carpetas genéricas `helpers`, `utils` o `common` salvo que exista una responsabilidad compartida real y claramente nombrable.

### Aplicación al histórico de vehículos

`vehicle-history.tool.ts` ya concentra validación, transformación, fechas, catálogo y respuesta pública, por lo que se aprueba un refactor controlado hacia:

```text
tools/vehicle-history.tool.ts
contracts/vehicle-history.contract.ts
services/vehicle-history.service.ts
repositories/vehicle-history.repository.ts
repositories/vehicle-history.cursor.ts
catalogs/vehicle-events.catalog.ts
```

El refactor debe preservar exactamente el comportamiento y contrato existente.


---

## DEC-025 — Errores MCP sin structuredContent

**Estado:** Aprobada

Las respuestas exitosas de las Tools pueden incluir `structuredContent` validado contra su `outputSchema`.

Las respuestas de error con `isError: true` no incluirán `structuredContent`; el error se entregará mediante `content` como texto estructurado.

### Motivo

Evitar que clientes MCP validen una estructura de error contra el `outputSchema` definido para respuestas exitosas.

Los errores funcionales conservan su código y mensaje, por ejemplo:

```text
INVALID_PLATES
INVALID_DATE_RANGE
INVALID_CURSOR
DATA_SOURCE_ERROR
```


---

## DEC-026 — Streamable HTTP stateless para la V1

**Estado:** Aprobada

La V1 expone el MCP remoto mediante Streamable HTTP en modalidad stateless.

Implementación:

```text
createMcpHandler(factory, { legacy: "stateless" })
        ↓
toNodeHandler(...)
        ↓
node:http
```

El factory crea un `McpServer` por petición, mientras el repository y el pool PostgreSQL se reutilizan.

No se incorporan sesiones propias, session store, event store, Redis ni resumabilidad en la V1.

---

## DEC-027 — Adaptador oficial Node para Streamable HTTP

**Estado:** Aprobada

Se utiliza:

```text
@modelcontextprotocol/node@2.1.0
```

como adaptador oficial para `node:http`.

La dependencia es compatible con `@modelcontextprotocol/server@2.2.0`.

Aunque el adaptador incluye Hono como dependencia transitiva, Proing MCP no utiliza Hono directamente ni adopta Hono como framework de aplicación.

---

## DEC-028 — HTTP local restringido a localhost durante el MVP

**Estado:** Aprobada

Durante el MVP local, Streamable HTTP escucha únicamente en:

```text
127.0.0.1
```

y aplica validación de Host y Origin mediante las utilidades oficiales del adaptador Node.

El proceso Node no implementa autenticación, OAuth, TLS ni CORS genérico, y no
se expone en `0.0.0.0`. En producción, HTTPS termina en Nginx.

Para el despliegue productivo detrás de Nginx, el bind de Node se conserva en
`127.0.0.1`. Las validaciones usan `hostHeaderValidation(...)` y
`originValidation(...)` con listas explícitas configuradas mediante:

```text
HTTP_ALLOWED_HOSTS
HTTP_ALLOWED_ORIGINS
```

Sin configuración, los defaults continúan limitados a `localhost`,
`127.0.0.1` y `[::1]`. En producción se agrega explícitamente el hostname
público, sin esquema ni puerto. No se permiten comodines ni se reemplazan las
validaciones por CORS genérico. Origin sigue siendo opcional para clientes MCP
no-browser; cuando está presente debe estar autorizado.


---

## DEC-029 — Tamaño de página por defecto orientado a consumibilidad

**Estado:** Aprobada

`operacion.consultar_historico_vehiculos` utiliza:

```text
DEFAULT_PAGE_SIZE = 100
MAX_PAGE_SIZE = 5000
```

El cambio de 1000 a 100 registros por defecto se basa en una prueba E2E real
con Claude Code por stdio. Una página de 1000 registros produjo aproximadamente
271.886 caracteres y no pudo ser consumida directamente por el cliente. Una
página de 250 registros produjo aproximadamente 67.741 caracteres y también
fue desviada a un archivo auxiliar. Con 100 registros por página, el agente
consumió directamente todas las páginas y recuperó correctamente 1.402
registros en 15 llamadas MCP, siguiendo `next_cursor` hasta `has_more=false`.

El ajuste no cambia la paginación keyset, la estructura del cursor ni el máximo
permitido. La metadata pública debe explicar cómo continuar páginas y cómo
dividir períodos superiores a 31 días. Para consultas grandes, debe desalentar
el aumento de `limit` como mecanismo para recuperar todo el conjunto en una
sola respuesta y orientar al agente a usar el valor por defecto con
`next_cursor` mientras `has_more=true`.


---

## DEC-030 — Bearer token estático individual por usuario para el Security MVP

**Estado:** Aprobada e implementada

El Streamable HTTP productivo requiere un Bearer token estático individual por usuario:

```http
Authorization: Bearer <token_usuario>
```

Cada token se genera con 32 bytes aleatorios mediante CSPRNG. El servidor almacena únicamente su SHA-256, asociado a un `userId` y al indicador `enabled`, en un archivo de configuración externo al repositorio.

Formato versionado:

```json
{
  "version": 1,
  "users": [
    {
      "id": "usuario",
      "tokenHash": "<sha256>",
      "enabled": true
    }
  ]
}
```

La ruta se configura mediante:

```text
MCP_AUTH_TOKENS_FILE
```

y es obligatoria únicamente para el transporte HTTP. `stdio` permanece sin autenticación.

### Punto de aplicación

La autenticación se ejecuta después de las validaciones de Host y Origin y antes de `toNodeHandler`:

```text
/mcp
 ↓
Host
 ↓
Origin
 ↓
TokenAuthenticator
 ↓
AuthContext
 ↓
toNodeHandler
 ↓
MCP
```

Una request no autenticada no puede alcanzar `initialize`, `tools/list`, `tools/call`, las capas de dominio ni PostgreSQL.

### Identidad

Una autenticación exitosa crea un contexto mínimo por request:

```text
userId
authType = static_token
```

El transporte lo adapta al `AuthInfo` del SDK. El token original puede existir transitoriamente en memoria durante la request, pero nunca se persiste ni se registra.

### Motivo

La solución permite cerrar rápidamente el endpoint productivo, revocar usuarios individualmente y disponer de identidad básica sin introducir todavía un proveedor OAuth, sesiones propias o una base de datos de autorización.

SHA-256 directo es aceptable en este caso porque los tokens no son passwords humanos sino secretos CSPRNG de 256 bits.

### Operación

El archivo de tokens se carga y valida una sola vez antes de que HTTP comience a escuchar. Una modificación de tokens requiere reinicio ordenado del servicio. Hot reload queda fuera de alcance.

Los fallos de autenticación devuelven de forma uniforme:

```text
401 Unauthorized
```

sin revelar si el token es inexistente o está deshabilitado.

### Alcance

Esta decisión no implementa autorización diferenciada. Mientras esté vigente:

```text
usuario autenticado
        ↓
mismas Tools publicadas
```

OAuth, RBAC, permisos por Tool y data scopes quedan para una fase posterior antes de incorporar capacidades con niveles de sensibilidad diferentes.

### Evidencia E2E productiva

Se validó en producción:

- sin token → `401 Unauthorized`;
- token inválido → `401 Unauthorized`;
- token válido → autenticación superada;
- `initialize` → exitoso;
- `tools/list` → exitoso;
- `tools/call` → exitoso;
- caso de control `TUZ64G`, 28-Sep-2026 → 66 registros;
- Claude Desktop usando exclusivamente el conector productivo autenticado → 66 registros.


---

## DEC-031 — Administración local segura de tokens mediante CLI

**Estado:** Aprobada e implementada

La administración de usuarios y tokens del Security MVP se realizará mediante una CLI local versionada con el proyecto:

```text
src/security/admin/auth-cli.ts
```

Comandos soportados:

```text
create <userId>
rotate <userId>
disable <userId>
enable <userId>
list
```

### Reglas

- `create` y `rotate` generan 32 bytes aleatorios con CSPRNG y muestran el token original una sola vez;
- sólo se persiste SHA-256;
- `disable` y `enable` modifican únicamente el estado;
- `list` muestra únicamente usuario y estado;
- el formato de `userId` es `^[a-z][a-z0-9._-]{0,63}$`;
- la CLI reutiliza el mismo schema y archivo `MCP_AUTH_TOKENS_FILE`;
- las escrituras son atómicas y utilizan lock local;
- la CLI no ejecuta `systemctl` ni implementa hot reload;
- las mutaciones requieren reinicio manual del servicio.

### Separación de privilegios

En producción:

```text
/etc/proing-mcp                 root:proing-mcp 0750
/etc/proing-mcp/tokens.json     root:proing-mcp 0640
```

El proceso `proing-mcp` tiene lectura pero no escritura sobre sus propias credenciales. Las mutaciones se ejecutan mediante `sudo` por un operador autorizado.

### Motivo

Evitar generación de hashes y edición manual de JSON, reducir riesgo de corrupción del archivo y conservar una operación simple sin introducir una base de datos, UI administrativa, endpoint remoto, OAuth ni privilegios adicionales para el proceso MCP.

### Evidencia

La implementación fue validada con 130/130 tests y build exitoso. En producción se verificó el flujo:

```text
create usuario
   ↓
reinicio
   ↓
token autentica
   ↓
disable usuario
   ↓
reinicio
   ↓
mismo token recibe 401
```
