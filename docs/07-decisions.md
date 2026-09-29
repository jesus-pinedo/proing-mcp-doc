# 07 — Decisiones

Registro inicial de decisiones arquitectónicas del proyecto.

---

## DEC-001 — Node.js + TypeScript

**Estado:** Aprobada

El servidor MCP inicial se desarrollará con Node.js y TypeScript.

### Motivo

- ecosistema natural para MCP;
- tipado;
- facilidad de evolución;
- familiaridad con el stack;
- despliegue liviano.

---

## DEC-002 — PostgreSQL directo desde el MCP

**Estado:** Aprobada

La primera versión se conectará directamente a PostgreSQL.

No se creará inicialmente una capa de servicios HTTP intermedia exclusivamente para que el MCP consuma información.

### Motivo

Crear servicios adicionales introduciría más componentes y trabajo sin aportar valor suficiente para el MVP.

La opción podrá revisarse si posteriormente otros consumidores necesitan reutilizar esos servicios.

---

## DEC-003 — La lógica compleja de datos no vive en el agente

**Estado:** Aprobada

Cuando una consulta requiera múltiples tablas, joins o reglas de negocio, esa lógica deberá resolverse antes de entregar la información al agente.

Se priorizarán:

- funciones PostgreSQL;
- vistas;
- consultas controladas en la capa de datos.

El agente trabajará con capacidades y filtros de negocio.

---

## DEC-004 — Arquitectura por dominios

**Estado:** Aprobada

El MCP crecerá organizado por dominios.

Ejemplos:

- operación;
- inventario.

Los Tools utilizarán nombres como:

`operacion.consultar_historico`

`inventario.consultar_existencia`

---

## DEC-005 — Primer MVP local

**Estado:** Aprobada

El primer MCP se ejecutará localmente y deberá poder ser consumido por un agente/cliente MCP compatible.

La infraestructura remota se decidirá después de validar el patrón.

---

## DEC-006 — Documentación separada temporalmente

**Estado:** Aprobada

Durante la etapa inicial, la documentación puede mantenerse en este repositorio independiente del código.

El código se trabajará localmente con Codex.

Posteriormente la documentación podrá migrarse al repositorio definitivo de GitLab junto con el proyecto.

---

## DEC-007 — No exponer tablas como interfaz del producto

**Estado:** Aprobada

Aunque algunos casos iniciales puedan depender de una única tabla histórica, el patrón general será exponer capacidades de negocio.

Esto permite que el mismo enfoque funcione cuando un dominio dependa de muchas tablas.
