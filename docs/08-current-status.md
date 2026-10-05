# 08 — Estado actual

## Fase

**Bloques 0–10 cerrados / listo para iniciar Bloque 11 — medición de rendimiento**

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
- Streamable HTTP previsto para HTTP/local-remoto.
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
- La Tool soportará una o varias placas y hasta 31 días por consulta.
- El histórico devolverá coordenadas, dirección, velocidad y evento normalizado.
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

## Pendientes inmediatos

1. Ejecutar Bloque 11: medición de rendimiento con los escenarios documentados.
2. Medir antes de modificar índices o arquitectura.
3. Mantener `DEFAULT_PAGE_SIZE=100` y `MAX_PAGE_SIZE=5000`.
4. No crear índices hasta contar con evidencia de las mediciones.

## Bloque 10 — Resultado

**Estado: CERRADO**

La validación E2E real con Claude Code por stdio confirmó:

- descubrimiento del servidor y de la Tool;
- invocación real contra PostgreSQL;
- paginación por cursor y múltiples páginas;
- `INVALID_DATE_RANGE`, `INVALID_CURSOR` e `INVALID_PLATES`;
- consultas de múltiples placas;
- normalización de timezone.

Hallazgo de consumibilidad:

- 1000 registros produjeron aproximadamente 271.886 caracteres y la respuesta
  no pudo ser consumida directamente por Claude Code;
- con `limit=100`, Claude Code recuperó 1.402 registros en 15 llamadas MCP,
  siguiendo `next_cursor` hasta `has_more=false`.

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
- `localhostHostValidation()`;
- `localhostOriginValidation()`;
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
- índice existente `(tso_placa, tso_fecha_hora)` utilizado por PostgreSQL;
- 1 placa × 1 día: ~1.36 ms;
- 1 placa × 30 días: ~1.18 s en primera lectura con I/O y ~12 ms en caché;
- no se requieren índices adicionales para el MVP.

La paginación futura utilizará cursor lógico basado en:

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

- autenticación remota definitiva;
- autorización por usuario;
- alta disponibilidad;
- múltiples dominios completos;
- interfaz gráfica;
- persistencia propia del MCP;
- prompts personales persistidos;
- escritura directa sobre PostgreSQL desde Tools;
- SQL arbitrario generado por LLM.

## Próximo hito

**Ejecutar el Bloque 11: medición de rendimiento.**
