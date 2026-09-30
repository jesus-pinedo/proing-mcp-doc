# 08 — Estado actual

## Fase

**Definición cerrada de arquitectura base / preparación del MVP técnico**

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

## Pendientes inmediatos

1. Confirmar la zona horaria real almacenada en `tso_fecha_hora`.
2. Cerrar la definición de `mcp.vw_historico_vehiculos`.
3. Definir valores iniciales de page size, máximo de placas y timeout.
4. Validar rendimiento con consultas reales y `EXPLAIN ANALYZE`.
5. Crear el proyecto local Node.js + TypeScript.
6. Instalar y configurar el SDK MCP.
7. Crear usuario/permisos PostgreSQL.
8. Implementar conexión y pool.
9. Implementar catálogo JSON de eventos.
10. Implementar la primera Tool.
11. Probarla desde un cliente MCP compatible.

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

**Cerrar zona horaria, superficie PostgreSQL y límites operativos de `operacion.consultar_historico_vehiculos` para dejar la V1 lista para implementación por Codex.**
