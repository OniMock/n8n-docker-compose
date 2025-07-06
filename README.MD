# WhatsApp Automation Stack

This project sets up a Docker Compose stack for automating WhatsApp workflows using n8n, Redis, PostgreSQL, and WAHA (WhatsApp HTTP API).

## Overview

The stack includes:

- **Redis**: In-memory data store for caching.
- **PostgreSQL**: Relational database for persistent storage.
- **WAHA**: WhatsApp HTTP API for sending/receiving messages.
- **n8n**: Workflow automation tool for orchestrating processes.

## Prerequisites

- Docker and Docker Compose installed.
- `.env` file with required environment variables (see below).

## Environment Variables

Create a `.env` file in the project root with the following variables:

```env
# Redis
REDIS_PASSWORD=your_redis_password

# PostgreSQL
POSTGRES_USER=your_postgres_user
POSTGRES_PASSWORD=your_postgres_password
POSTGRES_DB=your_postgres_db

# WAHA
N8N_URL=http://your_n8n_host:5678
WHATSAPP_HOOK_URL=${N8N_URL}/webhook/webhook
WHATSAPP_DEFAULT_ENGINE=GOWS
WHATSAPP_HOOK_EVENTS=message

# n8n
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=your_n8n_user
N8N_BASIC_AUTH_PASSWORD=your_n8n_password
WEBHOOK_URL=${N8N_URL}
GENERIC_TIMEZONE=America/Sao_Paulo
N8N_LOG_LEVEL=debug
```

Replace placeholders (e.g., `your_redis_password`, `your_n8n_host`) with actual values.

## Services

### Redis

- **Image**: `redis:latest`
- **Platform**: `linux/amd64`
- **Configuration**: Password-protected with `REDIS_PASSWORD`.
- **Volume**: `redis_data` for persistent data storage at `/data`.

### PostgreSQL

- **Image**: `postgres:latest`
- **Platform**: `linux/amd64`
- **Configuration**: Uses `POSTGRES_USER`, `POSTGRES_PASSWORD`, and `POSTGRES_DB` for setup.
- **Volume**: `pgdata` for persistent database storage at `/var/lib/postgresql/data`.

### WAHA

- **Image**: `devlikeapro/waha:latest`
- **Platform**: `linux/amd64`
- **Configuration**:
  - Connects to n8n via `WHATSAPP_HOOK_URL`.
  - Uses `GOWS` engine for WhatsApp API.
  - Listens for `message` events.
- **Volumes**:
  - `waha_sessions`: Stores session data at `/app/.sessions`.
  - `waha_media`: Stores media files at `/app/.media`.

### n8n

- **Image**: `n8nio/n8n:latest`
- **Platform**: `linux/amd64`
- **Configuration**:
  - Basic auth enabled with `N8N_BASIC_AUTH_USER` and `N8N_BASIC_AUTH_PASSWORD`.
  - Webhook URL set to `N8N_URL`.
  - Timezone set to `America/Sao_Paulo`.
  - Log level set to `debug` for detailed logs.
- **Volume**: `n8n_data` for persistent workflow data at `/home/node/.n8n`.

## Volumes

- `pgdata`: PostgreSQL data.
- `redis_data`: Redis data.
- `waha_sessions`: WAHA session data.
- `waha_media`: WAHA media files.
- `n8n_data`: n8n workflow data.

## Usage

1. Clone this repository or create the `docker-compose.yml` file.
2. Create and configure the `.env` file with the required variables.
3. Run the stack:
   ```bash
   docker-compose up -d
   ```
4. Access n8n at `http://your_n8n_host:5678` to configure workflows.
5. Use WAHA to interact with WhatsApp via its HTTP API.

## Stopping the Stack

To stop and remove containers:

```bash
docker-compose down
```

To remove volumes as well:

```bash
docker-compose down -v
```

## Notes

- Ensure `N8N_URL` is accessible from WAHA for webhook communication.
- Secure sensitive data in the `.env` file and avoid committing it to version control.
- Check container logs for debugging:
  ```bash
  docker-compose logs <service_name>
  ```

## Troubleshooting

- **Redis**: Verify `REDIS_PASSWORD` is set correctly.
- **PostgreSQL**: Ensure `POSTGRES_USER`, `POSTGRES_PASSWORD`, and `POSTGRES_DB` are valid.
- **WAHA**: Confirm `N8N_URL` is reachable and webhook events are configured.
- **n8n**: Check authentication credentials and logs for errors (`N8N_LOG_LEVEL=debug`).
