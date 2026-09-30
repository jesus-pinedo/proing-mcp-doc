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
