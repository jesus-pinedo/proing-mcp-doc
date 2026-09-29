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
- Primera Tool: `operacion.consultar_historico_vehiculo`.
- Resources y Prompts contemplados para evolución, pero fuera del alcance de la V1.
- Infraestructura productiva futura en una EC2 separada.
- Documentación temporalmente en repositorio independiente.
- Código a versionar en GitLab Proing.

## Pendientes inmediatos

1. Revisar la tabla histórica real y sus campos.
2. Definir el contrato funcional de `operacion.consultar_historico_vehiculo`.
3. Definir filtros requeridos y opcionales.
4. Definir límites de fecha, registros, paginación y timeout.
5. Diseñar `mcp.vw_historico_vehiculos` o decidir si una consulta controlada es suficiente.
6. Definir el contrato de salida.
7. Crear el proyecto local Node.js + TypeScript.
8. Instalar y configurar el SDK MCP.
9. Crear usuario/permisos PostgreSQL.
10. Implementar conexión y pool.
11. Implementar la primera Tool.
12. Probarla desde un cliente MCP compatible.

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

**Contrato funcional aprobado para `operacion.consultar_historico_vehiculo`, listo para implementación por Codex.**
