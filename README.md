# Starter Kit — Odoo with Traefik (staging / production)

Docker Compose stack with integrated Traefik reverse proxy, automatic TLS via Let's Encrypt, and longpolling support. Addons are bundled in the image — no local mount needed.

## Prerequisites

- Docker and Docker Compose installed on the server
- A domain name pointing to the server's IP (A or CNAME record)
- Ports 80 and 443 open on the firewall

## Structure

```
.
├── docker-compose.yml
├── .env                  # Environment variables (do not commit)
└── config/
    └── odoo.conf         # Odoo configuration (mounted read-only)
```

## Configuration

### 1. `.env` file

Copy `.env.example` to `.env` and fill in the values:

```bash
cp .env.example .env
```

| Variable                 | Description                                            |
| ------------------------ | ------------------------------------------------------ |
| `DOMAIN`                 | Main domain for the instance (e.g. `odoo.example.com`) |
| `ACME_EMAIL`             | Email for Let's Encrypt notifications                  |
| `ODOO_DB_USER`           | PostgreSQL user                                        |
| `ODOO_DB_PASSWORD`       | PostgreSQL password                                    |
| `TRAEFIK_DASHBOARD_AUTH` | BasicAuth hash for the Traefik dashboard               |

> **Important**: `$` characters in the htpasswd hash must be escaped as `$$` in the `.env` file.
> Docker Compose interprets `$xxx` as an interpolation variable.

To generate the hash:
```bash
htpasswd -nb admin yourpassword
# Then replace every $ with $$ in the .env file
```

### 2. `config/odoo.conf`

Key settings for this environment:

- `proxy_mode = True` — required behind Traefik
- `workers = 4` — adjust based on available CPUs (rule of thumb: 2 × CPU + 1)
- `max_cron_threads = 0` — disabled by default, enable after validating the restored database (see Restore section)
- `longpolling_port = 8072` — dedicated port for the real-time bus (notifications, chatter)

## Start

```bash
docker compose up -d
```

Traefik provisions the TLS certificate on first start. The instance is then available at `https://${DOMAIN}`.

The Traefik dashboard is available at `https://traefik.${DOMAIN}` (BasicAuth protected).

## Connect to the instance

### Odoo container shell

```bash
docker compose exec odoo bash
```

### Odoo shell (Python REPL)

```bash
docker compose exec odoo odoo shell --no-http -d <database_name>
```

## Restore a backup

> Before restoring, make sure `max_cron_threads = 0` in `odoo.conf` to prevent scheduled actions
> from running on the database being loaded.
> Also neutralize outgoing emails and external connectors
> (built-in option available since Odoo 16).

Restoration can be done via the [Database Manager](http://localhost:8069/web/database/manager) or via CLI.

See the full documentation: [Operations > Database Management](https://apikcloud.github.io/docs/#/17-database).

## Stop

```bash
docker compose down
```

Add `-v` to also remove volumes (data will be lost).

## Architecture notes

- Odoo does not expose any port directly on the host — all traffic goes through Traefik.
- Longpolling (`/websocket`, `/longpolling`) is routed to port `8072` via a dedicated Traefik router — required for the real-time bus to work.
- PostgreSQL is not reachable from outside; Odoo waits for the Postgres healthcheck to pass before starting.

## Resources

- [Odoo Documentation](https://www.odoo.com/documentation)
- [Traefik Documentation](https://doc.traefik.io/traefik/)
- [Apik Docs](https://apikcloud.github.io/docs/)
