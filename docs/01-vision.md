# 01 — Visión

## Propósito

Construir un servidor **MCP (Model Context Protocol)** para Proing que permita a agentes de IA consultar información corporativa de forma controlada, entendible y reutilizable.

La primera versión busca validar el patrón técnico con un alcance pequeño antes de crecer hacia múltiples dominios.

## Problema

Las aplicaciones internas de Proing contienen información distribuida en múltiples tablas, relaciones y reglas de negocio. Exponer directamente esas tablas a un agente produciría:

- bajo entendimiento semántico;
- dependencia del modelo físico de datos;
- riesgo de consultas incorrectas;
- mayor superficie de seguridad;
- dificultad para evolucionar la base de datos.

Por ello, el MCP no expondrá tablas arbitrarias. Expondrá **capacidades de negocio**.

## Principio central

El agente solicita una capacidad mediante un Tool.

El MCP valida los parámetros y delega la obtención de la información a una interfaz estable, preferiblemente una función o vista preparada en PostgreSQL.

```text
Agente
  ↓
MCP Tool
  ↓
Capa de acceso a datos
  ↓
Vista / función PostgreSQL
  ↓
Tablas internas
```

## Alcance inicial

Primera capacidad objetivo:

`operacion.consultar_historico`

Debe permitir consultar el histórico asociado a operación/vehículos utilizando filtros definidos y retornando información ya preparada para consumo del agente.

## Evolución esperada

El MCP podrá crecer por dominios, por ejemplo:

- Operación
- Inventario
- Documentación
- Otros dominios internos

Cada dominio deberá exponer capacidades de negocio y no el esquema físico completo.

## Objetivo del MVP técnico

Demostrar que un agente compatible con MCP, como ChatGPT, Codex, Claude o Gemini cuando el cliente lo soporte, puede:

1. descubrir una herramienta;
2. enviar filtros;
3. ejecutar la consulta de manera controlada;
4. recibir datos estructurados;
5. razonar sobre los resultados sin conocer la estructura interna de la base de datos.
