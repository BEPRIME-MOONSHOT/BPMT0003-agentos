# Despliegue de producción en Coolify

Este repositorio conserva `compose.yaml` y `compose.prod.yaml` para desarrollo/local. Para Coolify usa el archivo único:

```text
docker-compose.coolify.yml
```

## Crear el recurso

1. En Coolify crea una **Application** desde el repositorio Git `BEPRIME-MOONSHOT/BPMT0003-agentos`, rama `main`.
2. Selecciona **Docker Compose** y define el archivo Compose como `/docker-compose.coolify.yml`.
3. Configura el dominio en el servicio `agentos-api`, con puerto interno `8000`.
4. Mantén PostgreSQL sin dominio ni puertos publicados; solo existe en la red interna de Compose.

## Variables obligatorias de Coolify

Guárdalas como variables/secretos de Coolify. No las confirmes en Git.

```dotenv
OPENAI_API_KEY=<secret>
DB_PASS=<secret aleatorio largo>
AGENTOS_URL=https://agentos.example.com
JWT_VERIFICATION_KEY="-----BEGIN PUBLIC KEY-----
...clave pública de os.agno.com...
-----END PUBLIC KEY-----"
```

Para generar valores:

```sh
openssl rand -base64 32 # DB_PASS
openssl rand -base64 32 # MCP_CONNECT_SECRET
```

Opcionales:

```dotenv
MCP_CONNECT_SECRET=<secret de 32+ caracteres>
PARALLEL_API_KEY=<secret>
SLACK_BOT_TOKEN=<secret>
SLACK_SIGNING_SECRET=<secret>
ENABLE_DEPLOY_CHECK=True
DB_USER=ai
DB_DATABASE=ai
```

`JWT_VERIFICATION_KEY` es obligatorio en producción. En os.agno.com: **Connect OS → Live**, usa el dominio público, activa Token-Based Authorization (JWT) y copia su clave pública.

## Validación posterior al despliegue

```sh
curl -i https://agentos.example.com/health
# esperado: HTTP 200

curl -i https://agentos.example.com/agents
# esperado: HTTP 401 sin bearer JWT
```

## Sincronización con Agno upstream

`.github/workflows/sync-upstream.yml` corre cada lunes a las 08:17 UTC y también puede lanzarse manualmente. Detecta cambios de `agno-agi/agentos-docker:main`, actualiza `sync/agno-upstream` y abre/actualiza un PR a `main`.

El workflow nunca despliega ni fusiona automáticamente. Configura Coolify para desplegar solamente al recibir cambios en `main`, después de revisar y fusionar el PR.
