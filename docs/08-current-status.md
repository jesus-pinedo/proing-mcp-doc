# 08 — Estado actual

## Fase

**Bloques 0 y 1 cerrados / listo para iniciar Bloque 2 del MVP técnico**

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

1. Ejecutar Bloque 2: configuración tipada y pool PostgreSQL.
2. Crear `.env` local con credenciales de desarrollo.
3. Validar conexión con `SELECT 1`.
4. Implementar cierre ordenado del pool.
5. Dejar preparado el reemplazo posterior por el usuario definitivo `proing_mcp`.
6. No consultar todavía el histórico desde una Tool.

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

**Ejecutar el Bloque 2 del plan de implementación: configuración y conexión PostgreSQL.**
