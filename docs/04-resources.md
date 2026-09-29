# 04 — Resources

## Definición

Los **Resources** proporcionan contexto consultable por los clientes MCP.

A diferencia de un Tool, un Resource no representa principalmente una operación; representa información contextual reutilizable.

## Candidatos iniciales

### Catálogos de operación

`proing://operacion/catalogos/estados`

Puede contener estados y significados utilizados en operación.

### Bodegas

`proing://inventario/bodegas`

Puede exponer un catálogo controlado de bodegas cuando el dominio de inventario sea incorporado.

### Documentación de procesos

`proing://documentacion/procesos`

Puede servir como punto de entrada para contexto documental seleccionado.

## Criterio de uso

Usar Resource cuando:

- la información es principalmente contextual;
- puede reutilizarse en múltiples consultas;
- no necesita ejecutar una acción de negocio;
- su lectura ayuda al agente a interpretar correctamente los datos.

Usar Tool cuando:

- hay filtros;
- hay una consulta operativa;
- se ejecuta lógica;
- el resultado depende de parámetros enviados por el agente.

## Estado

Los Resources quedan definidos conceptualmente para la arquitectura, pero no son obligatorios para demostrar el primer MVP.
