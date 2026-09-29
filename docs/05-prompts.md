# 05 — Prompts

## Definición

Los **Prompts MCP** son plantillas reutilizables que un cliente compatible puede presentar al usuario para iniciar tareas frecuentes.

No sustituyen el prompt libre del usuario.

## Candidatos

### analizar_operacion_vehiculo

Objetivo:

Guiar el análisis de la operación de un vehículo usando las capacidades disponibles en el MCP.

### generar_reporte_inventario

Objetivo futuro:

Orientar al agente para consultar y resumir información de inventario.

## Tipos de prompts

Se distinguen dos conceptos:

### Prompts corporativos

Plantillas definidas y publicadas por el MCP.

Ejemplo:

`analizar_operacion_vehiculo`

### Prompts personales

Instrucciones o plantillas que cada usuario pueda guardar en el cliente o en una solución superior.

Estas no necesariamente pertenecen al servidor MCP.

## Estrategia inicial

Los Prompts no son prioritarios para la primera prueba técnica.

El MVP debe demostrar primero:

1. conexión cliente ↔ MCP;
2. descubrimiento de Tools;
3. ejecución de una consulta;
4. respuesta estructurada.

Posteriormente se incorporarán Prompts cuando aporten valor real a los usuarios clave.
