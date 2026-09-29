# 06 — Seguridad

## Objetivo

Definir desde el inicio límites claros para el acceso de agentes de IA a información interna.

## Principios

### Menor privilegio

El usuario de PostgreSQL utilizado por el MCP debe tener únicamente los permisos necesarios.

Idealmente:

- lectura;
- ejecución de funciones autorizadas;
- acceso limitado a vistas específicas.

### Sin SQL libre

El agente no debe enviar SQL arbitrario para ser ejecutado por el MCP.

### Contratos explícitos

Cada Tool debe definir:

- parámetros permitidos;
- tipos;
- validaciones;
- límites;
- errores controlados.

### Secretos fuera del repositorio

Las credenciales deben estar en variables de entorno o un sistema de secretos.

No deben versionarse:

- passwords;
- tokens;
- certificados privados;
- connection strings con credenciales.

## Primera versión local

Aunque el MVP sea local, debe mantenerse separación entre:

```text
Código
Configuración
Secretos
```

Ejemplo conceptual:

```text
.env
DATABASE_HOST=
DATABASE_PORT=
DATABASE_NAME=
DATABASE_USER=
DATABASE_PASSWORD=
```

El archivo `.env` no debe almacenarse en Git.

## Seguridad futura

Antes de publicar el MCP remotamente se deberá definir:

- autenticación;
- autorización;
- TLS;
- exposición de red;
- auditoría;
- rate limits;
- trazabilidad por usuario;
- alcance por dominio o Tool.

## Protección de datos

No se debe asumir que por estar disponible en PostgreSQL una información debe exponerse al agente.

Cada nuevo Tool deberá pasar por una revisión explícita de datos y permisos.
