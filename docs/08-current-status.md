# 08 — Estado actual

## Fase

**Bloques 0, 1, 2 y 3 cerrados / listo para iniciar Bloque 4 del MVP técnico**

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

1. Ejecutar Bloque 4: catálogo JSON de eventos.
2. Crear `src/domains/operacion/catalogs/eventos-vehiculo.json`.
3. Implementar normalización de eventos conocidos, desconocidos y vacíos.
4. Mantener sin cambios la View PostgreSQL.
5. No implementar todavía repository, cursor, Tool ni transportes.

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

**Ejecutar el Bloque 4: catálogo JSON de eventos.**
