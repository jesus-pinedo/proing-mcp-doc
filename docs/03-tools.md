# 03 — Tools

## Definición

Los **Tools** representan acciones o consultas que el agente puede ejecutar mediante MCP.

Deben modelar capacidades de negocio, no tablas.

## Convención

Se utilizará la forma:

`dominio.accion`

Ejemplos:

- `operacion.consultar_historico_vehiculo`
- `operacion.consultar_ordenes`
- `inventario.consultar_existencia`

## Tool inicial

### operacion.consultar_historico_vehiculo

Estado: **por definir / implementar en MVP**

Objetivo:

Consultar el histórico de un vehículo mediante filtros explícitos de negocio.

Responsabilidades del Tool:

- recibir parámetros definidos;
- validar tipos y límites;
- invocar la capa de datos;
- retornar un resultado estructurado;
- controlar errores;
- evitar que el agente conozca el modelo físico de PostgreSQL.

La Tool no debe contener SQL libre enviado por el agente.

## Contrato preliminar

El contrato exacto se definirá al revisar la fuente real de datos.

Ejemplo conceptual:

```json
{
  "placa": "ABC123",
  "fecha_desde": "2026-01-01T00:00:00",
  "fecha_hasta": "2026-01-31T23:59:59"
}
```

La estructura anterior es ilustrativa y no constituye todavía el contrato definitivo.

El documento funcional de esta Tool deberá definir:

- objetivo;
- casos de uso;
- parámetros requeridos;
- parámetros opcionales;
- límites;
- origen de datos;
- View/Function utilizada;
- campos de salida;
- paginación;
- manejo de fechas;
- ordenamiento;
- errores;
- ejemplos;
- criterios de aceptación.

## Tools futuros candidatos

### operacion.consultar_ordenes

Consulta controlada de órdenes operativas mediante filtros de negocio.

### operacion.consultar_productividad

Consulta analítica de productividad sobre una superficie de datos preparada.

### inventario.consultar_existencia

Consulta de existencia por material, bodega u otros filtros definidos.

## Regla de diseño

Antes de crear un Tool nuevo se debe responder:

1. ¿Representa una necesidad de negocio entendible por el usuario?
2. ¿Puede definirse un contrato estable?
3. ¿Evita exponer detalles internos innecesarios?
4. ¿Existe una fuente confiable para obtener la información?
5. ¿Requiere Tool, Resource o simplemente contexto documental?
6. ¿Es lectura/reporting o ejecuta una acción de negocio?

Para lectura/reporting podrá acceder directamente a una View, Function o consulta controlada de PostgreSQL.

Para acciones de negocio se preferirá utilizar la API o servicio del dominio.
