# 08 — Estado actual

## Fase

**Definición inicial / preparación del MVP técnico**

## Ya definido

- MCP con Node.js + TypeScript.
- Primera ejecución local.
- Conexión directa del MCP a PostgreSQL.
- Sin base de datos propia para el MCP.
- Arquitectura orientada a dominios.
- Tools como capacidades de negocio.
- La complejidad de joins y reglas debe resolverse fuera del agente.
- Uso preferente de funciones o vistas para consultas complejas.
- Primer dominio: Operación.
- Primer Tool candidato: `operacion.consultar_historico`.
- Resources y Prompts contemplados desde arquitectura, pero no obligatorios para la primera prueba.
- Documentación temporalmente en repositorio independiente.
- Código trabajado localmente con Codex.

## Pendientes inmediatos

1. Crear el proyecto local Node.js + TypeScript.
2. Definir estructura inicial de carpetas.
3. Instalar y configurar el SDK MCP.
4. Revisar la fuente de datos real para `operacion.consultar_historico`.
5. Definir los filtros exactos del Tool.
6. Definir el contrato de respuesta.
7. Crear usuario/permisos de PostgreSQL apropiados.
8. Implementar la conexión.
9. Implementar el primer Tool.
10. Probarlo desde un cliente MCP.

## Fuera de alcance por ahora

- autenticación remota;
- despliegue productivo;
- alta disponibilidad;
- múltiples dominios completos;
- interfaz gráfica;
- persistencia propia del MCP;
- servicios HTTP adicionales;
- prompts personales persistidos.

## Próximo hito

**MCP local funcional capaz de responder correctamente una consulta de histórico de operación desde un agente compatible.**
