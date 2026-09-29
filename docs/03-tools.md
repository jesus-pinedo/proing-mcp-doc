# 03 — Tools

## Definición

Los **Tools** representan acciones o consultas que el agente puede ejecutar mediante MCP.

Deben modelar capacidades de negocio, no tablas.

## Convención

Se utilizará la forma:

`dominio.accion`

Ejemplos:

- `operacion.consultar_historico`
- `operacion.consultar_ordenes`
- `inventario.consultar_existencia`

## Tool inicial

### operacion.consultar_historico

Estado: **por definir / implementar en MVP**

Objetivo:

Consultar información histórica de operación a partir de filtros explícitos.

Responsabilidades del Tool:

- recibir parámetros definidos;
- validar los filtros;
- invocar la capa de datos;
- retornar un resultado estructurado;
- controlar errores y límites.

El Tool no debe contener joins extensos ni conocimiento innecesario del modelo físico.

## Contrato preliminar

El esquema exacto se definirá al revisar la tabla histórica y los filtros requeridos.

Ejemplo conceptual:

```json
{
  "vehiculo": "ABC123",
  "fecha_desde": "2026-01-01",
  "fecha_hasta": "2026-01-31"
}
```

La estructura anterior es ilustrativa y no constituye todavía el contrato definitivo.

## Tools futuros candidatos

### operacion.consultar_ordenes

Consulta controlada de órdenes operativas mediante filtros de negocio.

### inventario.consultar_existencia

Consulta de existencia por material, bodega u otros filtros definidos.

## Regla de diseño

Antes de crear un Tool nuevo se debe responder:

1. ¿Representa una necesidad de negocio entendible por el usuario?
2. ¿Puede definirse un contrato estable?
3. ¿Evita exponer detalles internos innecesarios?
4. ¿Existe una fuente confiable para obtener la información?
5. ¿Requiere Tool, Resource o simplemente contexto documental?
