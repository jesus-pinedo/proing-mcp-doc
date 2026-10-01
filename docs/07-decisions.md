# 07 — Decisiones

Registro de decisiones arquitectónicas aprobadas para Proing MCP.

---

## DEC-001 — Node.js + TypeScript

**Estado:** Aprobada

El servidor MCP inicial se desarrollará con Node.js LTS y TypeScript.

Se utilizará el SDK oficial de MCP para TypeScript, validación con Zod y módulos ESM.

### Motivo

- ecosistema natural para MCP;
- tipado;
- facilidad de evolución;
- familiaridad con el stack;
- despliegue liviano.

---

## DEC-002 — PostgreSQL directo desde el MCP para lectura/reporting

**Estado:** Aprobada

La V1 se conectará directamente a PostgreSQL mediante un cliente `pg` y un pool compartido.

No se creará inicialmente una capa HTTP intermedia exclusivamente para envolver consultas de lectura.

### Motivo

Crear endpoints adicionales para consultas que únicamente consume el MCP introduce componentes y mantenimiento sin aportar suficiente valor al MVP.

---

## DEC-003 — API/servicio para acciones de negocio

**Estado:** Aprobada

Las operaciones que registren, modifiquen información o ejecuten procesos de negocio deberán preferentemente utilizar la API o servicio del dominio correspondiente.

### Motivo

Las reglas, transacciones y validaciones empresariales no deben duplicarse dentro del MCP.

---

## DEC-004 — La lógica compleja de datos no vive en el agente

**Estado:** Aprobada

Cuando una consulta requiera múltiples tablas, JOIN, agregaciones o reglas, esa complejidad deberá resolverse antes de entregar la información al agente.

Se priorizarán:

- Views PostgreSQL;
- Functions PostgreSQL;
- consultas controladas en la capa de datos.

El agente trabajará con capacidades y filtros de negocio.

---

## DEC-005 — Schema de exposición para MCP

**Estado:** Aprobada

Se propone utilizar un schema dedicado, conceptualmente `mcp`, para Views y Functions preparadas para consumo del MCP.

### Motivo

Permite separar el modelo físico de Proing de la superficie de datos autorizada para agentes.

---

## DEC-006 — Arquitectura por dominios

**Estado:** Aprobada

El MCP crecerá organizado por dominios.

Ejemplos:

- operación;
- inventario;
- personal.

Los Tools utilizarán nombres orientados a capacidades, por ejemplo:

`operacion.consultar_historico_vehiculos`

`inventario.consultar_existencia`

---

## DEC-007 — Un único MCP modular inicialmente

**Estado:** Aprobada

La V1 utilizará un solo servidor Proing MCP.

No se crearán servidores separados por dominio únicamente por aumentar la cantidad de Tools.

Se reevaluará cuando existan fronteras reales de seguridad, equipo, infraestructura, despliegue o escala.

---

## DEC-008 — Primer MVP local

**Estado:** Aprobada

El primer MCP se ejecutará localmente y deberá poder ser consumido por un cliente MCP compatible.

El núcleo será independiente del transporte.

---

## DEC-009 — stdio local y Streamable HTTP remoto

**Estado:** Aprobada

Durante el desarrollo local se utilizará principalmente `stdio`.

La arquitectura incluirá Streamable HTTP para pruebas y futuro despliegue remoto.

Las Tools no dependerán del transporte.

---

## DEC-010 — Primera Tool de histórico de vehículo

**Estado:** Aprobada

La primera capacidad será:

`operacion.consultar_historico_vehiculos`

Su contrato funcional se definirá antes de implementar.

---

## DEC-011 — Resources y Prompts no forman parte de la V1

**Estado:** Aprobada

La arquitectura contempla Tools, Resources y Prompts, pero la primera versión implementará únicamente la Tool necesaria.

Resources y Prompts se incorporarán cuando exista una necesidad concreta.

---

## DEC-012 — Usuario PostgreSQL dedicado y read-only

**Estado:** Aprobada

El MCP utilizará un usuario de base de datos exclusivo con principio de mínimo privilegio.

La V1 será exclusivamente de lectura.

---

## DEC-013 — Sin SQL generado por el agente

**Estado:** Aprobada

No se expondrá una capacidad genérica para ejecutar SQL suministrado por el LLM.

El agente únicamente suministrará filtros de negocio definidos por cada contrato.

---

## DEC-014 — Infraestructura productiva separada

**Estado:** Aprobada

Cuando se despliegue en AWS, el Proing MCP se ejecutará en una instancia separada de la EC2 actual de aplicaciones.

### Motivo

Evitar acoplar el nuevo artefacto a una instancia que ya ejecuta Apache y otros procesos, y reservar una frontera clara para futuras capacidades de IA/integración.

---

## DEC-015 — Documentación separada temporalmente

**Estado:** Aprobada

Durante la etapa inicial, la documentación se mantiene en este repositorio independiente.

El código se trabajará localmente y posteriormente se versionará en GitLab Proing.

La documentación podrá migrarse al repositorio definitivo cuando se decida consolidar ambos.


---

## DEC-016 — Catálogo de eventos en JSON para el MVP

**Estado:** Aprobada

El catálogo que normaliza los valores de `tso_evento` se mantendrá inicialmente como un archivo JSON versionado dentro del dominio de Operación del MCP.

Ubicación conceptual:

```text
src/domains/operacion/catalogs/eventos-vehiculo.json
```

PostgreSQL conservará y entregará el valor original del proveedor. La capa de dominio del MCP traducirá ese valor a código, nombre y descripción Proing.

### Motivo

- catálogo pequeño y estable;
- simplicidad para el MVP;
- versionado en Git;
- evita crear una tabla adicional únicamente para este artefacto;
- permite migrarlo posteriormente a una fuente dinámica sin cambiar el contrato de la Tool.


---

## DEC-017 — Zona horaria del histórico

**Estado:** Aprobada

La columna `transportes.tr_vehiculos_tso_historico.tso_fecha_hora`, definida como `timestamp without time zone`, se interpretará en la V1 como hora local de Colombia:

```text
America/Bogota
UTC-05:00
```

### Evidencia

En registros recientes, `tso_fecha_hora` y `tso_fecha_server` utilizan el mismo reloj local y normalmente presentan diferencias de segundos o pocos minutos, sin un desfase sistemático de cinco horas.

Se observó al menos un registro con hora del dispositivo adelantada respecto a la hora del servidor. Este comportamiento se considera una posible anomalía del dispositivo/proveedor, no un cambio de zona horaria.

### Regla de contrato

El MCP interpretará `tso_fecha_hora` como `America/Bogota` y expondrá fechas con offset explícito.

---

## DEC-018 — Credenciales locales mediante .env

**Estado:** Aprobada

Las credenciales reales de PostgreSQL para desarrollo local se configurarán en un archivo `.env` no versionado.

El repositorio incluirá únicamente `.env.example` sin secretos.

El usuario definitivo `proing_mcp` podrá crearse posteriormente; al estar disponible, sus credenciales sustituirán las credenciales temporales de desarrollo en el `.env`.

Variables mínimas:

```text
DATABASE_HOST
DATABASE_PORT
DATABASE_NAME
DATABASE_USER
DATABASE_PASSWORD
APP_TIMEZONE=America/Bogota
```


---

## DEC-019 — Cursor basado en fecha_hora + placa

**Estado:** Aprobada

La paginación de `operacion.consultar_historico_vehiculos` utilizará como clave lógica:

```text
fecha_hora
placa
```

No se incluirá `id_interno` en el cursor.

### Motivo

La tabla fuente posee una restricción única sobre:

```text
(tso_placa, tso_fecha_hora)
```

Por lo tanto, la combinación lógica `(fecha_hora, placa)` identifica de forma determinística cada registro consultable y permite simplificar el cursor.

---

## DEC-020 — No crear índice adicional para el MVP

**Estado:** Aprobada

No se crearán índices adicionales para la primera versión.

### Evidencia

El índice existente:

```text
UNIQUE (tso_placa, tso_fecha_hora)
```

fue utilizado correctamente por PostgreSQL en las pruebas.

Resultados observados:

```text
1 placa × 1 día:
Execution Time ≈ 1.36 ms

1 placa × 30 días:
primera lectura con bloques desde disco ≈ 1.18 s
segunda lectura con bloques en caché ≈ 12 ms
```

El costo principal observado en la primera lectura de 30 días provino de I/O de disco, no de la búsqueda por índice.

### Regla

No se agregará un índice nuevo hasta que el uso real o futuras mediciones demuestren una necesidad concreta.


---

## DEC-021 — Fecha histórica como texto local en la capa de datos

**Estado:** Aprobada

El repository de histórico obtiene `fecha_hora` desde PostgreSQL como texto local explícito con microsegundos:

```text
YYYY-MM-DDTHH:mm:ss.ffffff
```

No se crea un objeto JavaScript `Date` a partir del `timestamp without time zone` de PostgreSQL.

### Motivo

Evitar conversiones implícitas de zona horaria dependientes de Node.js, sistema operativo o configuración del proceso.

La capa pública de la Tool será responsable de añadir el offset aprobado de Colombia (`-05:00`) al serializar la respuesta MCP.


---

## DEC-022 — INVALID_CURSOR como error funcional

**Estado:** Aprobada

La Tool `operacion.consultar_historico_vehiculos` incorpora el código de error:

```text
INVALID_CURSOR
```

para cursores corruptos, estructuralmente inválidos o no soportados.

El error debe ser controlado y no exponer detalles internos de codificación.

---

## DEC-023 — Schema estricto de salida para la Tool

**Estado:** Aprobada

La primera Tool define y valida también su contrato público de salida mediante Zod.

### Motivo

- evitar fugas accidentales de campos internos;
- mantener estable el contrato MCP;
- detectar desviaciones entre repository, transformación de dominio y respuesta pública;
- facilitar pruebas automatizadas del contrato.

Esto no implica crear una capa genérica de schemas para futuras Tools; cada Tool podrá definir únicamente lo necesario.


---

## DEC-024 — Separación Tool / Contract / Service / Repository

**Estado:** Aprobada

Para Tools con suficiente complejidad, Proing MCP utilizará la siguiente separación de responsabilidades:

```text
Tool
 ↓
Contract
 ↓
Service
 ↓
Repository
```

### Responsabilidades

- **Tool**: adaptación al protocolo MCP, metadata, registro, invocación y traducción de errores.
- **Contract**: schemas Zod públicos de entrada/salida y tipos derivados.
- **Service**: orquestación del caso de uso, normalización de fechas, composición de respuesta y uso de catálogos.
- **Repository**: persistencia, SQL, paginación y acceso a fuentes de datos.

### Regla

No se crearán Controllers adicionales para las Tools MCP: la Tool ya representa la frontera equivalente al controller/adaptador.

No todas las Tools requieren Service; se introduce cuando existe lógica de aplicación suficiente para separar responsabilidades.

No se utilizarán carpetas genéricas `helpers`, `utils` o `common` salvo que exista una responsabilidad compartida real y claramente nombrable.

### Aplicación al histórico de vehículos

`vehicle-history.tool.ts` ya concentra validación, transformación, fechas, catálogo y respuesta pública, por lo que se aprueba un refactor controlado hacia:

```text
tools/vehicle-history.tool.ts
contracts/vehicle-history.contract.ts
services/vehicle-history.service.ts
repositories/vehicle-history.repository.ts
repositories/vehicle-history.cursor.ts
catalogs/vehicle-events.catalog.ts
```

El refactor debe preservar exactamente el comportamiento y contrato existente.


---

## DEC-025 — Errores MCP sin structuredContent

**Estado:** Aprobada

Las respuestas exitosas de las Tools pueden incluir `structuredContent` validado contra su `outputSchema`.

Las respuestas de error con `isError: true` no incluirán `structuredContent`; el error se entregará mediante `content` como texto estructurado.

### Motivo

Evitar que clientes MCP validen una estructura de error contra el `outputSchema` definido para respuestas exitosas.

Los errores funcionales conservan su código y mensaje, por ejemplo:

```text
INVALID_PLATES
INVALID_DATE_RANGE
INVALID_CURSOR
DATA_SOURCE_ERROR
```
