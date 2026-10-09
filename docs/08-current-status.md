# 08 — Estado actual

## Fase

**MVP técnico cerrado / Security MVP con autenticación HTTP desplegada y validada en producción**

## Ya definido

- Un único servidor Proing MCP modular por dominios.
- Node.js LTS + TypeScript.
- SDK MCP para TypeScript.
- Zod para validación.
- `pg` para PostgreSQL.
- Proyecto ESM.
- Primera ejecución local.
- Núcleo independiente del transporte.
- `stdio` como transporte principal local.
- Streamable HTTP stateless implementado para uso local y evolución remota.
- Conexión directa del MCP a PostgreSQL para lectura/reporting.
- Pool PostgreSQL compartido.
- Sin base de datos propia para el MCP.
- Schema conceptual `mcp` para Views/Functions de exposición.
- Arquitectura orientada a dominios.
- Tools como capacidades de negocio.
- La complejidad de JOIN y reglas debe resolverse fuera del agente.
- Uso de Views, Functions o consultas controladas según corresponda.
- Acciones de negocio futuras preferentemente mediante APIs/servicios.
- Usuario PostgreSQL dedicado y read-only.
- Sin SQL generado por el agente.
- Primer dominio: Operación.
- Primera Tool: `operacion.consultar_historico_vehiculos`.
- La Tool soporta una o varias placas y hasta 31 días por consulta.
- El histórico devuelve coordenadas, dirección, velocidad y evento normalizado.
- El catálogo de eventos del MVP vivirá en `src/domains/operacion/catalogs/eventos-vehiculo.json`.
- La normalización del evento se realizará en la capa de dominio del MCP, conservando el valor original del proveedor.
- Resources y Prompts contemplados para evolución, pero fuera del alcance de la V1.
- Infraestructura productiva futura en una EC2 separada.
- Documentación temporalmente en repositorio independiente.
- Código a versionar en GitLab Proing.
- `tso_fecha_hora` se interpretará como hora Colombia (`America/Bogota`, UTC-05:00).
- La conectividad PostgreSQL de desarrollo fue validada.
- El usuario de desarrollo actual tiene permiso `SELECT` sobre la tabla histórica.
- Las credenciales definitivas del futuro usuario `proing_mcp` se configurarán mediante `.env` local no versionado.
- Límites finales: `DEFAULT_PAGE_SIZE=100`, `MAX_PAGE_SIZE=5000`,
  `MAX_DATE_RANGE_DAYS=31`, `APP_TIMEZONE=America/Bogota`.
- Arquitectura implementada: Tool → Contract → Service → Repository →
  PostgreSQL → `mcp.vw_historico_vehiculos` → tabla histórica.
- Endpoint productivo: `https://mcp.proing.com.co/mcp`.
- HTTPS y Nginx productivos operativos.
- Streamable HTTP productivo protegido mediante Bearer token individual por usuario.
- Tokens generados con 256 bits aleatorios; el servidor conserva únicamente SHA-256.
- Configuración externa mediante `MCP_AUTH_TOKENS_FILE=/etc/proing-mcp/tokens.json`.
- Gate de autenticación aplicado antes de `toNodeHandler` y del handler MCP.
- Identidad mínima por request: `userId` + `authType=static_token`.
- `stdio` continúa funcionando sin autenticación.
- Tests posteriores a la administración operativa de tokens: 130/130 aprobados.
- `npm run build`: exitoso.
- E2E productivo con Claude Desktop autenticado: `TUZ64G`, 28-Sep-2026 → 66 registros.

## Pendientes posteriores al MVP

1. Incorporar auditoría estructurada por usuario y Tool.
2. Diseñar autorización por roles/permisos y filtrado de `tools/list` / protección de `tools/call` antes de publicar Tools con sensibilidad diferenciada.
3. Incorporar data scopes cuando una Tool deba limitar registros según usuario, contrato, regional u otro alcance.
4. Evolucionar posteriormente a OAuth/identidad corporativa para interoperabilidad y gobierno de acceso a mayor escala.
5. Revisar rate limiting y configuración de logging de Nginx para asegurar que `Authorization` nunca se registre.
6. Medir nuevamente antes de modificar índices, cursor o arquitectura.
7. Evaluar nuevas Tools únicamente a partir de necesidades de negocio aprobadas.

## Security MVP — Autenticación HTTP por usuario

**Estado: CERRADO EN PRODUCCIÓN**

Implementación:

- `src/security/auth-context.ts`;
- `src/security/token-authenticator.ts`;
- `src/security/token-repository.ts`;
- configuración `MCP_AUTH_TOKENS_FILE`;
- archivo de tokens versionado mediante `version: 1`;
- SHA-256 de tokens CSPRNG de 256 bits;
- comparación mediante `timingSafeEqual`;
- usuario habilitado/deshabilitado mediante `enabled`;
- respuesta uniforme `401 Unauthorized`;
- protección de todos los métodos sobre `/mcp`;
- carga de credenciales una sola vez antes de `listen()`;
- sin nuevas dependencias;
- sin cambios en Tool, Contract, Service, Repository o PostgreSQL;
- `stdio` preservado.

Validación automática:

```text
npm test      → 116/116
npm run build → exitoso
```

Validación productiva:

```text
sin token       → 401
token inválido  → 401
token válido    → acceso al protocolo MCP
initialize      → OK
tools/list      → OK
tools/call      → OK
```

Caso E2E de control:

```text
cliente: Claude Desktop
conector: Proing MCP Producción
Tool: operacion.consultar_historico_vehiculos
placa: TUZ64G
fecha: 28-Sep-2026
resultado: 66 registros
has_more: false
next_cursor: null
```

El conector de Claude utiliza `Sin inicio de sesión` y un Request Header fijo:

```text
Authorization: Bearer <token_usuario>
```

OAuth, permisos por Tool, RBAC y data scopes permanecen fuera de este bloque.

---

## Security MVP — Administración operativa de tokens

**Estado: CERRADO EN PRODUCCIÓN**

Implementación:

- CLI única en `src/security/admin/auth-cli.ts`;
- comandos `create`, `rotate`, `disable`, `enable` y `list`;
- tokens nuevos generados con `randomBytes(32)` y base64url;
- SHA-256 como único valor persistido;
- validación compartida con `TokenRepository`;
- escritura atómica mediante temporal + `rename`;
- lock local para mutaciones concurrentes;
- preservación de permisos/propietario del archivo;
- sin hot reload ni reinicio automático;
- separación de privilegios: proceso `proing-mcp` sólo lectura, mutaciones mediante `sudo`.

Comando productivo base:

```bash
sudo env \
  MCP_AUTH_TOKENS_FILE=/etc/proing-mcp/tokens.json \
  /usr/bin/node \
  /opt/proing-mcp/app/dist/security/admin/auth-cli.js \
  <create|rotate|disable|enable|list> [userId]
```

Validación automática:

```text
npm test      → 130/130
npm run build → exitoso
```

Validación productiva:

```text
list                  → jesus enabled
create mcp-test       → token generado
reinicio              → token mcp-test autentica
disable mcp-test      → estado disabled
reinicio              → mismo token devuelve 401
```

Después de cualquier mutación se requiere:

```bash
sudo systemctl restart proing-mcp
```

---

## Bloque 12 — Resultado

**Estado: CERRADO**

La documentación fue reconciliada con la implementación real y consolida:

- arquitectura y transportes implementados;
- contrato, límites, paginación y fechas finales;
- errores y hallazgos de interoperabilidad;
- validación E2E con Claude Code por `stdio`;
- resultados de rendimiento del Bloque 11;
- suficiencia de índices y cursor para el MVP;
- pendientes explícitamente posteriores al MVP.

## Bloque 11 — Resultado

**Estado: CERRADO**

Se ejecutó `EXPLAIN (ANALYZE, BUFFERS)` dos veces por escenario, sin crear
índices, cambiar configuración ni ejecutar mantenimiento.

### Primera página

| Escenario | Filas/trabajo principal | Plan | Execution Time | Shared read |
|---|---|---|---:|---:|
| `TUZ64G` × 1 día | 66 filas | Index Scan + quicksort 41 kB | 0,95–1,91 ms | 0 |
| `TUZ64G` × 30 días | 1.402 candidatos, 101 retornados | Bitmap Index/Heap Scan + top-N 46 kB | 6,19–13,32 ms | 0 |
| 5 placas × 30 días | 33.398 examinadas, 33.296 descartadas | Index Scan por fecha + Incremental Sort | 31,39–41,43 ms | 0 |
| 10 placas × 30 días | 6.312 examinadas, 6.210 descartadas | Index Scan por fecha + Incremental Sort | 5,70–10,62 ms | 0 |

Las cinco placas sumaron 6.932 registros; las diez placas, 13.670. La prueba de
diez placas fue más rápida que la de cinco por una mayor densidad cronológica de
coincidencias, confirmando que el costo no crece linealmente con la cantidad de
placas.

### Páginas posteriores por cursor

Para `TUZ64G`, septiembre de 2026:

| Página | Filas procesadas | Execution Time | Buffers hit |
|---|---:|---:|---:|
| Primera | 1.402 | 6,56–6,73 ms | 1.361 |
| Intermedia, tras 700 registros | 702 | 3,36–3,37 ms | 686 |
| Final, tras 1.400 registros | 2 | 0,13–0,15 ms | 6 |

La condición keyset se incorpora al `Index Cond`, evita volver a recorrer filas
anteriores y no presenta degradación tipo `OFFSET`. Los índices actuales y el
cursor son suficientes para el MVP; no se justifica agregar un índice.

## Bloque 10 — Resultado

**Estado: CERRADO**

La validación E2E real con Claude Code por stdio confirmó:

- conexión MCP, `tools/list`, descubrimiento del servidor y de la Tool;
- metadata y anotación `readOnly`;
- invocación real contra PostgreSQL;
- resultado vacío;
- paginación por cursor y múltiples páginas;
- `INVALID_DATE_RANGE`, `INVALID_CURSOR` e `INVALID_PLATES`;
- consultas de múltiples placas;
- normalización de timezone;
- interpretación del resultado por el modelo.

Hallazgo de consumibilidad:

- 1000 registros produjeron aproximadamente 271.886 caracteres y la respuesta
  no pudo ser consumida directamente por Claude Code;
- 250 registros produjeron aproximadamente 67.741 caracteres y la respuesta
  también fue desviada a un archivo auxiliar;
- con `limit=100`, Claude Code recuperó 1.402 registros en 15 llamadas MCP,
  siguiendo `next_cursor` hasta `has_more=false`.

El caso de referencia produjo 14 páginas de 100 registros y una última página
de 2, con `has_more=false` y `next_cursor=null`. El usuario solicitó el
histórico completo sin conocer paginación ni cursores; Claude Code siguió la
metadata automáticamente.

Ajuste de cierre:

- `DEFAULT_PAGE_SIZE` cambió de 1000 a 100;
- `MAX_PAGE_SIZE` permanece en 5000;
- metadata y schemas públicos explican el rango máximo de 31 días, continuidad
  por cursor y normalización a `America/Bogota` (`UTC-05:00`);
- no cambiaron cursor, Repository, SQL, View, catálogo ni transportes;
- la diferencia de formato entre errores de validación y errores funcionales
  queda registrada sin cambios en este bloque.

## Bloque 9 — Resultado

**Estado: CERRADO**

Auditoría final de cobertura realizada.

Resultado:

- tests antes: 78;
- tests después: 82;
- 0 fallidos;
- no se modificó código funcional;
- se reforzaron límites de paginación;
- se agregó rechazo explícito de campos adicionales;
- se reforzó validación de cursor;
- se fortaleció integración MCP completa en memoria;
- se demostró factory-per-request en HTTP con repository compartido;
- metadata de Tool validada incluyendo `title` y `readOnlyHint`;
- `npm run build`: exitoso;
- `npm test`: 82 aprobados, 0 fallidos;
- `npm run db:check`: exitoso;
- entrypoints stdio y HTTP confirmados.

No se detectaron bugs funcionales.

## Bloque 8 — Resultado

**Estado: CERRADO**

Implementación reportada:

- `src/transports/http.ts`;
- `src/transports/lifecycle.ts`;
- reutilización del lifecycle desde `stdio`;
- `@modelcontextprotocol/node@2.1.0`;
- `createMcpHandler(..., { legacy: "stateless" })`;
- `toNodeHandler(...)`;
- `hostHeaderValidation(...)` con allowlist configurable;
- `originValidation(...)` con allowlist configurable;
- servidor nativo `node:http`;
- escucha exclusiva en `127.0.0.1`;
- endpoint único `/mcp`;
- rutas diferentes → 404;
- pool PostgreSQL singleton compartido;
- `createProingServer()` reutilizado por request;
- sin sesiones MCP propias;
- cierre HTTP idempotente;
- `npm run mcp:http` agregado;
- `npm run build`: exitoso;
- `npm test`: 78 aprobados, 0 fallidos;
- `npm run db:check`: exitoso;
- `tools/list` por HTTP: exitoso;
- `tools/call` real por HTTP: exitoso;
- `stdio` continúa funcionando.

Habilitación productiva del transporte HTTP:

- `HTTP_ALLOWED_HOSTS` y `HTTP_ALLOWED_ORIGINS` aceptan listas CSV de hostnames;
- ambas usan por defecto `localhost`, `127.0.0.1` y `[::1]`;
- una variable definida no puede quedar sin entradas válidas;
- no se aceptan comodines;
- `mcp.proing.com.co` debe agregarse explícitamente en producción;
- Host u Origin presentes y no autorizados continúan devolviendo 403;
- requests sin Origin continúan permitidos para clientes MCP no-browser;
- Node permanece en `127.0.0.1` detrás de Nginx;
- no se agregó CORS genérico ni se cambió la arquitectura de dominio.

Configuración productiva actual:

```env
HTTP_ALLOWED_HOSTS=localhost,127.0.0.1,mcp.proing.com.co
HTTP_ALLOWED_ORIGINS=localhost,127.0.0.1,mcp.proing.com.co
```

## Bloque 7.5 — Resultado

**Estado: CERRADO**

Refactor implementado:

```text
src/domains/operacion/
├── tools/
│   └── vehicle-history.tool.ts
├── contracts/
│   └── vehicle-history.contract.ts
├── services/
│   └── vehicle-history.service.ts
├── repositories/
│   ├── vehicle-history.repository.ts
│   └── vehicle-history.cursor.ts
└── catalogs/
    ├── eventos-vehiculo.json
    └── vehicle-events.catalog.ts
```

Responsabilidades finales:

- Tool: adaptación MCP, metadata, registro, invocación del Service y traducción de errores.
- Contract: schemas Zod públicos, límites contractuales y tipos derivados.
- Service: caso de uso, fechas Colombia, catálogo, repository y construcción de respuesta.
- Repository: SQL, parámetros, cursor y paginación.
- Catalog: conocimiento estático de eventos.
- Transport: conexión MCP.

Validaciones:

- `AGENTS.md` actualizado con la regla permanente;
- tests antes: 70;
- tests después: 72;
- `npm run build`: exitoso;
- `npm test`: 72 aprobados, 0 fallidos;
- `npm run db:check`: exitoso;
- `tools/list`: exitoso;
- invocación real exitosa con `limit=2`;
- `has_more=true` y `next_cursor` presentes;
- comportamiento MCP preservado;
- no se modificaron SQL, View, cursor, catálogo, stdio ni dependencias.

## Bloque 7 — Resultado

**Estado: CERRADO**

Implementado:

- servidor MCP reusable con `McpServer`;
- registro único de `operacion.consultar_historico_vehiculos`;
- transporte `StdioServerTransport`;
- lifecycle y cierre idempotente;
- protección de stdout;
- scripts de ejecución;
- `tools/list` validado con MCP Inspector 2.5.0;
- `npm run build`: exitoso;
- `npm test`: 68 aprobados, 0 fallidos;
- `npm run db:check`: exitoso.

Validación final:

- invocación real exitosa mediante MCP Inspector contra PostgreSQL;
- 66 registros reales devueltos para una consulta de un día;
- eventos normalizados correctamente;
- fechas con offset `-05:00`;
- coordenadas numéricas;
- respuesta exitosa validada contra `outputSchema`;
- cursor inválido devuelve `INVALID_CURSOR` con `isError: true`;
- respuestas de error ya no incluyen `structuredContent`;
- MCP Inspector reporta `tool_is_error` como comportamiento esperado para `isError: true`, sin error adicional de schema;
- `npm run build`: exitoso;
- `npm test`: 70 aprobados, 0 fallidos;
- `npm run db:check`: exitoso.

## Bloque 6 — Resultado

**Estado: CERRADO**

Implementación reportada:

- `vehicle-history.tool.ts`;
- schema Zod estricto de entrada;
- schema Zod estricto de salida;
- nombre público `operacion.consultar_historico_vehiculos`;
- metadata orientada a negocio;
- Tool marcada como read-only;
- normalización de entrada hacia `America/Bogota`;
- serialización de salida con offset `-05:00`;
- microsegundos preservados;
- integración exclusiva con `VehicleHistoryRepository`;
- normalización mediante el catálogo existente;
- errores `INVALID_PLATES`, `INVALID_DATE_RANGE`, `INVALID_CURSOR` y `DATA_SOURCE_ERROR`;
- `registerVehicleHistoryTool(server, dependencies)` preparado para el servidor;
- sin servidor, stdio ni HTTP;
- `npm run build`: exitoso;
- `npm test`: 60 aprobados, 0 fallidos.

## Bloque 5 — Resultado

**Estado: CERRADO**

Implementación reportada:

- `vehicle-history.repository.ts`;
- `vehicle-history.cursor.ts`;
- tests dedicados de repository y cursor;
- keyset pagination sin `OFFSET`;
- cursor opaco base64url versionado;
- cursor basado únicamente en `fecha_hora + placa`;
- SQL parametrizado;
- consulta exclusiva de `mcp.vw_historico_vehiculos`;
- orden `fecha_hora ASC, placa ASC`;
- patrón `LIMIT N + 1`;
- sin `COUNT(*)`;
- fecha histórica obtenida como texto local con microsegundos mediante `to_char`;
- sin conversión automática a `Date`/UTC;
- evento devuelto aún como valor original;
- repository testeable mediante una frontera mínima inyectable;
- `npm run build`: exitoso;
- `npm test`: 37 aprobados, 0 fallidos;
- no se avanzó a Tool ni transportes.

## Bloque 4 — Resultado

**Estado: CERRADO**

Implementación reportada:

- catálogo JSON con 27 eventos;
- `src/domains/operacion/catalogs/eventos-vehiculo.json`;
- resolver de dominio en `vehicle-events.catalog.ts`;
- lookup exacto y determinístico mediante `Map`;
- equivalencias `PANICO`, `PÁNICO` y `Panic button` → `PANICO / Pánico`;
- eventos desconocidos → `NO_CATALOGADO`;
- null/vacío/espacios → `SIN_EVENTO`;
- valor original preservado;
- descripciones vacías en V1;
- `resolveJsonModule` habilitado en TypeScript;
- sin dependencias nuevas;
- `npm run build`: exitoso;
- `npm test`: 18 aprobados, 0 fallidos;
- no se avanzó a repository, cursor, Tool ni transportes.

## Bloque 3 — Resultado

**Estado: CERRADO**

Se creó y validó:

```text
mcp.vw_historico_vehiculos
```

La View expone:

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

Validaciones realizadas:

- latitud convertida de forma segura y limitada a [-90, 90];
- longitud convertida de forma segura y limitada a [-180, 180];
- valores inválidos convertidos a NULL;
- `tso_fecha_server` no expuesto;
- evento conservado en su valor original;
- View creada correctamente en PostgreSQL;
- consulta real de 20 registros validada;
- índice existente `(tso_placa, tso_fecha_hora)` disponible para las consultas.

La medición final de índices y latencias está consolidada en el resultado del
Bloque 11. La paginación implementada utiliza un cursor lógico basado en:

```text
fecha_hora
placa
```

## Bloque 2 — Resultado

**Estado: CERRADO**

Implementación y validaciones reportadas:

- configuración tipada de variables de entorno;
- carga mediante `process.loadEnvFile(".env")`;
- sin dependencia `dotenv`;
- pool PostgreSQL singleton y reutilizable;
- `query_timeout` opcional;
- cierre explícito del pool;
- script `npm run db:check`;
- conexión PostgreSQL real validada con éxito;
- `npm run build`: exitoso;
- `npm test`: 7 aprobados, 0 fallidos;
- no se imprimieron secretos ni connection strings;
- no se avanzó al Bloque 3.

Dependencia de tipos agregada:

```text
@types/pg@8.23.1
```

## Bloque 1 — Resultado

**Estado: CERRADO**

Implementación reportada:

- Node.js `24.21.0`.
- npm `11.19.0`.
- `@modelcontextprotocol/server@2.2.0`.
- `zod@4.6.5`.
- `pg@8.23.0`.
- `typescript@7.0.2`.
- `tsx@4.23.15`.
- `@types/node@26.6.3`.
- proyecto ESM;
- build exitoso;
- ejecución compilada exitosa;
- test base aprobado;
- `.env` ignorado;
- `.env.example` creado sin secretos;
- estructura de carpetas base creada;
- no se adelantaron bloques posteriores.

## Fuera de alcance por ahora

- OAuth / autenticación corporativa definitiva;
- autorización por usuario, rol y Tool;
- data scopes por usuario o dominio;
- alta disponibilidad;
- múltiples dominios completos;
- interfaz gráfica;
- persistencia propia del MCP;
- prompts personales persistidos;
- escritura directa sobre PostgreSQL desde Tools;
- SQL arbitrario generado por LLM.

## Próximo hito

**Security MVP y administración operativa de tokens cerrados y validados en producción. Próximo foco: auditoría y autorización progresiva por Tool/usuario.**
