# 03 — Tools

## Definición

Los **Tools** representan acciones o consultas que el agente puede ejecutar mediante MCP.

Deben modelar capacidades de negocio, no tablas.

## Convención

Se utilizará la forma:

`dominio.accion`

Ejemplos:

- `operacion.consultar_historico_vehiculos`
- `operacion.consultar_ordenes`
- `inventario.consultar_existencia`

---

# Tool inicial

## operacion.consultar_historico_vehiculos

**Estado:** contrato funcional V1 en definición final.

### Objetivo

Consultar el histórico de una o varias placas dentro de un período determinado, entregando al agente una representación de negocio independiente del modelo físico de PostgreSQL.

La Tool debe permitir casos como:

- consultar el recorrido histórico de una o varias placas;
- analizar posiciones GPS;
- revisar velocidad;
- analizar eventos registrados;
- utilizar coordenadas para mapas, diagramas o análisis posteriores;
- contrastar el histórico con otras fuentes de datos cuando el volumen sea razonable.

La Tool recupera datos históricos. El análisis posterior corresponde al agente.

---

## Entrada V1

Ejemplo conceptual:

```json
{
  "placas": ["ABC123", "XYZ789"],
  "fecha_inicio": "2026-09-01T00:00:00-05:00",
  "fecha_fin": "2026-09-30T23:59:59-05:00",
  "limit": 100,
  "cursor": null
}
```

### Parámetros

| Campo | Tipo | Requerido | Descripción |
|---|---|---:|---|
| `placas` | `string[]` | Sí | Una o varias placas a consultar. Mínimo una placa. |
| `fecha_inicio` | datetime | Sí | Inicio del período con offset explícito. El rango hasta `fecha_fin` no puede superar 31 días. |
| `fecha_fin` | datetime | Sí | Fin del período con offset explícito. Debe ser igual o posterior a `fecha_inicio`; el rango no puede superar 31 días. |
| `limit` | integer | No | Registros por página. Valor por defecto: 100. Máximo: 5000. |
| `cursor` | string | No | Cursor opaco. Si `has_more=true`, debe recibir el `next_cursor` de la página anterior conservando placas y fechas. |

### Reglas iniciales

- rango máximo por consulta: **31 días**;
- para períodos mayores, el agente debe dividir la consulta en rangos de máximo 31 días;
- `limit` es opcional, con 100 registros por defecto y 5000 como máximo;
- si `has_more=true`, el agente debe continuar con `next_cursor` como `cursor`, conservando exactamente las mismas placas y rango de fechas;
- cuando necesite el conjunto completo, debe continuar hasta `has_more=false`;
- `registros[].fecha_hora` se devuelve en `America/Bogota` (`UTC-05:00`), independientemente del offset de entrada;
- la Tool debe soportar una o varias placas;
- el máximo de placas por solicitud será configurable y se ajustará según pruebas de volumen;
- las fechas relativas como “ayer” o “el mes pasado” deben ser resueltas por el agente antes de llamar la Tool;
- las placas deben normalizarse al formato esperado por Proing;
- no se acepta SQL enviado por el agente.

---

## Fuente de datos

Tabla operacional actual:

```text
transportes.tr_vehiculos_tso_historico
```

Campos disponibles:

```text
tso_id_
tso_placa
tso_fecha_hora
tso_latitud
tso_longitud
tso_direccion
tso_velocidad
tso_fecha_server
tso_evento
```

Índices relevantes existentes:

```text
PRIMARY KEY (tso_id_)
UNIQUE (tso_placa, tso_fecha_hora)
```

Se propone exponer esta información mediante una superficie controlada, preferiblemente:

```text
mcp.vw_historico_vehiculos
```

La View podrá encargarse de normalizar nombres y tipos sin exponer el esquema físico al agente.

La normalización del evento se realizará en la capa de dominio del MCP mediante un catálogo JSON versionado en el repositorio.

---

## Mapeo de datos

| PostgreSQL | Contrato MCP | Exposición |
|---|---|---|
| `tso_id_` | uso interno para cursor | No |
| `tso_placa` | `placa` | Sí |
| `tso_fecha_hora` | `fecha_hora` | Sí |
| `tso_latitud` | `latitud` | Sí |
| `tso_longitud` | `longitud` | Sí |
| `tso_direccion` | `direccion` | Sí |
| `tso_velocidad` | `velocidad` | Sí |
| `tso_fecha_server` | no definido para V1 | No |
| `tso_evento` | `evento.valor_origen` | Sí |

### Coordenadas

Aunque `tso_latitud` y `tso_longitud` están almacenadas como `varchar`, el contrato MCP debe devolver:

```text
number | null
```

Ejemplo:

```json
{
  "latitud": 3.4516,
  "longitud": -76.532
}
```

Si un valor no puede convertirse a coordenada válida, deberá devolverse `null` y no propagarse como texto inválido.

---

## Salida V1

Ejemplo conceptual:

```json
{
  "consulta": {
    "placas": ["ABC123", "XYZ789"],
    "fecha_inicio": "2026-09-01T00:00:00-05:00",
    "fecha_fin": "2026-09-30T23:59:59-05:00"
  },
  "paginacion": {
    "registros_retornados": 100,
    "has_more": true,
    "next_cursor": "..."
  },
  "registros": [
    {
      "placa": "ABC123",
      "fecha_hora": "2026-09-01T06:32:14-05:00",
      "latitud": 3.4516,
      "longitud": -76.532,
      "direccion": "Cali",
      "velocidad": 25.4,
      "evento": {
        "codigo": "ENCENDIDO",
        "nombre": "Encendido",
        "valor_origen": "Ignition ON",
        "descripcion": ""
      }
    }
  ]
}
```

### Campos por registro

| Campo | Tipo | Descripción |
|---|---|---|
| `placa` | string | Placa del vehículo. |
| `fecha_hora` | datetime | Fecha y hora del registro histórico. |
| `latitud` | number/null | Latitud normalizada. |
| `longitud` | number/null | Longitud normalizada. |
| `direccion` | string/null | Dirección reportada por el proveedor. |
| `velocidad` | number/null | Velocidad reportada. |
| `evento.codigo` | string | Código normalizado Proing. |
| `evento.nombre` | string | Nombre del evento en español. |
| `evento.valor_origen` | string/null | Valor original recibido del proveedor. |
| `evento.descripcion` | string | Descripción funcional. En V1 inicia vacía. |

---

# Catálogo inicial de eventos

El valor almacenado en `tso_evento` corresponde al evento recibido desde el proveedor.

El MCP no debe obligar al agente a interpretar directamente estos valores. Cada evento se normaliza a un código Proing y un nombre en español.

En la V1, este catálogo vivirá dentro del artefacto MCP como archivo JSON versionado:

```text
src/domains/operacion/catalogs/eventos-vehiculo.json
```

PostgreSQL entrega el valor original de `tso_evento`; la capa de dominio del MCP consulta el JSON y construye `evento.codigo`, `evento.nombre` y `evento.descripcion`.

La V1 inicia con el siguiente catálogo:

| valor_origen | codigo | nombre | descripcion |
|---|---|---|---|
| `Alerta de Inmovilidad` | `ALERTA_INMOVILIDAD` | Alerta de inmovilidad | |
| `Alta velocidad` | `ALTA_VELOCIDAD` | Alta velocidad | |
| `Battery Warning` | `ADVERTENCIA_BATERIA` | Advertencia de batería | |
| `Drive` | `EN_MOVIMIENTO` | En movimiento | |
| `End Speeding` | `FIN_EXCESO_VELOCIDAD` | Fin de exceso de velocidad | |
| `First Fix` | `PRIMERA_POSICION_GPS` | Primera posición GPS | |
| `Harsh Deceleration` | `DESACELERACION_BRUSCA` | Desaceleración brusca | |
| `Harsh Turn` | `GIRO_BRUSCO` | Giro brusco | |
| `Idle - Stop` | `FIN_RALENTI` | Fin de ralentí | |
| `Idle Alert` | `ALERTA_RALENTI` | Alerta de ralentí | |
| `Ignition OFF` | `APAGADO` | Apagado | |
| `Ignition ON` | `ENCENDIDO` | Encendido | |
| `Low Battery` | `BATERIA_BAJA` | Batería baja | |
| `PANICO` | `PANICO` | Pánico | |
| `POWER UP` | `ALIMENTACION_ENCENDIDA` | Alimentación encendida | |
| `Panic button` | `PANICO` | Pánico | |
| `Parked` | `ESTACIONADO` | Estacionado | |
| `Ping` | `PING` | Señal de comunicación | |
| `Power Cut` | `CORTE_ALIMENTACION` | Corte de alimentación | |
| `Power OFF` | `ALIMENTACION_APAGADA` | Alimentación apagada | |
| `Power Restored` | `ALIMENTACION_RESTAURADA` | Alimentación restablecida | |
| `PÁNICO` | `PANICO` | Pánico | |
| `Speed alert` | `ALERTA_VELOCIDAD` | Alerta de velocidad | |
| `Towing/Drifting?` | `REMOLQUE_DESPLAZAMIENTO` | Remolque o desplazamiento | |
| `Travel Start` | `INICIO_RECORRIDO` | Inicio de recorrido | |
| `Travel Stop` | `FIN_RECORRIDO` | Fin de recorrido | |
| `Unk Harsh Behavior` | `COMPORTAMIENTO_BRUSCO_DESCONOCIDO` | Comportamiento brusco no identificado | |

### Eventos equivalentes

Los siguientes valores de origen se normalizan al mismo concepto Proing:

```text
PANICO
PÁNICO
Panic button
      ↓
PANICO / Pánico
```

El valor original siempre se conserva en `evento.valor_origen` para trazabilidad.

### Evento no catalogado

La aparición de un nuevo valor del proveedor no debe causar error en la consulta.

Si `tso_evento` contiene un valor que aún no existe en el catálogo, la Tool devolverá:

```json
{
  "evento": {
    "codigo": "NO_CATALOGADO",
    "nombre": "Evento no catalogado",
    "valor_origen": "Nuevo evento proveedor",
    "descripcion": ""
  }
}
```

Esto permite mantener estable el contrato y ampliar el catálogo posteriormente.

### Descripción

En la V1 el campo `descripcion` permanecerá vacío para todos los eventos.

Posteriormente podrá documentarse el significado funcional Proing de cada evento sin cambiar la estructura del contrato.

---

## Paginación

La tabla histórica puede contener grandes volúmenes de información y la consulta debe soportar períodos de hasta un mes para varias placas.

Por lo tanto, la V1 deberá implementar paginación por cursor.

La respuesta incluirá:

```text
registros_retornados
has_more
next_cursor
```

No será obligatorio calcular un `COUNT(*)` total para cada consulta.

El tamaño por defecto es de 100 registros por página y el máximo permitido
permanece en 5000. Si una página devuelve `has_more=true`, el agente debe
repetir la misma consulta usando `next_cursor` como `cursor`, sin cambiar las
placas ni el rango de fechas. Para recuperar el conjunto completo debe continuar
hasta recibir `has_more=false`.

### Evidencia E2E de consumibilidad

La configuración inicial de 1000 registros produjo una respuesta de
aproximadamente 271.886 caracteres que Claude Code no pudo consumir
directamente. Con 100 registros por página, Claude Code recuperó correctamente
1.402 registros en 15 llamadas MCP siguiendo el cursor hasta
`has_more=false`. Por esta evidencia, 100 es el valor por defecto del contrato;
el máximo configurable continúa siendo 5000.

### Orden determinístico

Los registros se devolverán cronológicamente.

Orden conceptual:

```text
fecha_hora ASC
placa ASC
```

El cursor podrá utilizar internamente:

```text
fecha_hora
placa
```

pero estos detalles no deben quedar expuestos como parámetros editables para el usuario.

La combinación `(placa, fecha_hora)` es única en la tabla fuente, por lo que `id_interno` no es necesario como componente del cursor.

---

## Límites

La V1 deberá contemplar:

- máximo 31 días por consulta;
- límite de registros por página controlado por servidor;
- máximo de placas configurable;
- timeout de consulta;
- validación de fechas;
- consultas parametrizadas;
- usuario PostgreSQL read-only.

La paginación permite recorrer resultados grandes, pero no debe utilizarse como estrategia para transportar datasets masivos completos al LLM cuando posteriormente existan Tools agregadas más adecuadas.

Para el MVP se mantendrá una única Tool histórica y se evaluarán nuevos casos de uso a partir del uso real.

---

## Casos de error

### INVALID_DATE_RANGE

Se utiliza cuando:

- `fecha_inicio` es posterior a `fecha_fin`;
- el rango solicitado supera el máximo permitido.

### INVALID_PLATES

Se utiliza cuando:

- el array de placas está vacío;
- el formato de las placas no es válido;
- se supera el máximo de placas configurado.

### INVALID_CURSOR

Se utiliza cuando el cursor suministrado no puede decodificarse, tiene una estructura inválida o no corresponde a la versión soportada.

La respuesta debe ser controlada y no exponer detalles internos del cursor.

### DATA_SOURCE_ERROR

Error controlado de acceso a PostgreSQL.

No deberá exponer:

- SQL interno;
- stack traces;
- credenciales;
- nombres físicos innecesarios.

### Sin resultados

No se considera un error.

La respuesta deberá devolver:

```json
{
  "paginacion": {
    "registros_retornados": 0,
    "has_more": false,
    "next_cursor": null
  },
  "registros": []
}
```

---

## Regla de diseño general

Antes de crear una Tool nueva se debe responder:

1. ¿Representa una necesidad de negocio entendible por el usuario?
2. ¿Puede definirse un contrato estable?
3. ¿Evita exponer detalles internos innecesarios?
4. ¿Existe una fuente confiable para obtener la información?
5. ¿Requiere Tool, Resource o simplemente contexto documental?
6. ¿Es lectura/reporting o ejecuta una acción de negocio?

Para lectura/reporting podrá acceder directamente a una View, Function o consulta controlada de PostgreSQL.

Para acciones de negocio se preferirá utilizar la API o servicio del dominio.

---

## Tools futuros candidatos

Estos casos no forman parte del MVP y se evaluarán a partir del uso real:

- `operacion.consultar_ordenes`;
- `operacion.consultar_productividad`;
- Tool especializada para contrastar datasets grandes o archivos externos;
- `inventario.consultar_existencia`.
