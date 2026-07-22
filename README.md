# wp_blog — WordPress Docker Stack

A production-ready WordPress environment running on Docker Compose with Redis object caching, configured to connect to your host's global MySQL database and Adminer.

## Services

| Service | Image | Port |
|---|---|---|
| WordPress | `wordpress:latest` | [http://localhost:8080](http://localhost:8080) |
| Redis | `redis:7-alpine` | internal only |
| MySQL (Host) | VPS Global MySQL | `host.docker.internal:3306` |
| Adminer (Host) | VPS Global Adminer | Host managed |

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) or Docker Engine (includes Docker Compose v2)
- Existing MySQL instance running on host VPS

## Quick Start

```bash
# 1. Clone the repo
git clone https://github.com/elfedali/wordpress-docker.git
cd wordpress-docker

# 2. Create your local credentials file
cp .env.example .env

# 3. Open .env and set your host MySQL credentials
nano .env   # or: code .env

# 4. Start the stack
docker compose up -d

# 5. Open WordPress in your browser
open http://localhost:8080
```

## Environment Variables

All credentials are stored in `.env` (never committed). Use `.env.example` as the template.

| Variable | Description | Default |
|---|---|---|
| `WORDPRESS_DB_HOST` | Host MySQL address | `host.docker.internal:3306` |
| `WORDPRESS_DB_NAME` | Database name on host MySQL | `wp_blog` |
| `WORDPRESS_DB_USER` | WordPress DB user on host MySQL | `wp_blog` |
| `WORDPRESS_DB_PASSWORD` | WordPress DB password | `change_me_wp` |
| `WP_PORT` | Host port for WordPress | `8080` |

## Redis Object Cache

Redis is pre-configured in `wp-config.php` via `WORDPRESS_CONFIG_EXTRA`. To activate it:

1. Go to **WordPress Admin → Plugins → Add New**
2. Search for **Redis Object Cache** and install it
3. Click **Activate**, then **Enable Object Cache**

## Common Commands

```bash
# Start all services
docker compose up -d

# Stop all services (preserves data)
docker compose stop

# View logs (follow mode)
docker compose logs -f

# View logs for WordPress service
docker compose logs -f wordpress

# Restart a service
docker compose restart wordpress

# Tear down (removes containers; data volume is preserved)
docker compose down

# Tear down AND delete WordPress data volume (destructive!)
docker compose down -v
```

## Persistent Data

Data survives container restarts and removals via a named Docker volume:

| Volume | Contents |
|---|---|
| `wp_blog_data` | WordPress files (themes, plugins, uploads) |

To back up the WordPress uploads folder:

```bash
docker run --rm \
  -v wp_blog_data:/source \
  -v "$PWD":/backup \
  alpine tar czf /backup/wp_blog_data_backup.tar.gz -C /source .
```

## Project Structure

```
wp/
├── docker-compose.yml   # Stack definition
├── .env                 # Local credentials (git-ignored)
├── .env.example         # Credentials template (safe to commit)
├── php/
│   └── conf.d/
│       └── uploads.ini  # Custom PHP upload settings
├── .gitignore
└── README.md
```

## Security Notes

- **Change all default passwords** in `.env` before exposing any port to the internet.
- Do not commit `.env` — it is listed in `.gitignore`.
- Ensure host MySQL accepts connections from the Docker gateway (`host.docker.internal` / `172.17.0.1`).
- For production, place WordPress behind a reverse proxy (e.g., Nginx + Let's Encrypt / Traefik) and remove direct port mapping.

