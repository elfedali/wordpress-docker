# wp_blog — WordPress Docker Stack

A production-ready WordPress environment running on Docker Compose with MySQL, Redis object caching, and Adminer.

## Services

| Service | Image | Port |
|---|---|---|
| WordPress | `wordpress:latest` | [http://localhost:8080](http://localhost:8080) |
| Adminer (DB GUI) | `adminer:latest` | [http://localhost:8081](http://localhost:8081) |
| MySQL | `mysql:8.0` | internal only |
| Redis | `redis:7-alpine` | internal only |

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (includes Docker Compose v2)

## Quick Start

```bash
# 1. Clone the repo
git clone https://github.com/elfedali/wordpress-docker.git
cd wordpress-docker

# 2. Create your local credentials file
cp .env.example .env

# 3. Open .env and set strong passwords
nano .env   # or: code .env

# 4. Start the stack
docker compose up -d

# 5. Open WordPress in your browser
open http://localhost:8080
```

## Environment Variables

All credentials are stored in `.env` (never committed). Use `.env.example` as the template.

| Variable | Description |
|---|---|
| `MYSQL_ROOT_PASSWORD` | MySQL root password |
| `MYSQL_DATABASE` | Database name (`wp_blog`) |
| `MYSQL_USER` | WordPress DB user |
| `MYSQL_PASSWORD` | WordPress DB password |
| `WORDPRESS_DB_*` | Must match the MySQL values above |
| `WP_PORT` | Host port for WordPress (default `8080`) |
| `ADMINER_PORT` | Host port for Adminer (default `8081`) |

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

# View logs for a single service
docker compose logs -f wordpress

# Restart a service
docker compose restart wordpress

# Tear down (removes containers; data volumes are preserved)
docker compose down

# Tear down AND delete all data volumes (destructive!)
docker compose down -v
```

## Persistent Data

Data survives container restarts and removals via named Docker volumes:

| Volume | Contents |
|---|---|
| `wp_blog_db_data` | MySQL database files |
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
├── .gitignore
└── README.md
```

## Security Notes

- **Change all default passwords** in `.env` before exposing any port to the internet.
- Do not commit `.env` — it is listed in `.gitignore`.
- For production, place WordPress behind a reverse proxy (e.g., Nginx + Let's Encrypt / Traefik) and remove the direct port mapping.

## Upstream WordPress Docker Image (reference)

Quick reference: the official WordPress images are maintained by the Docker Community and documented on Docker Hub.

- **Where to get help:** Docker Community Slack, Server Fault, Unix & Linux, Stack Overflow.
- **Supported architectures:** amd64, arm32v5, arm32v6, arm32v7, arm64v8, i386, ppc64le, riscv64, s390x.

Example usage (docker run):

```bash
docker run --name some-wordpress --network some-network -d wordpress
```

Example compose snippet (upstream pattern):

```yaml
services:
  wordpress:
    image: wordpress
    restart: always
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: exampledb
    volumes:
      - wordpress:/var/www/html

  db:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
    volumes:
      - db:/var/lib/mysql

volumes:
  wordpress:
  db:
```

Notes from upstream:
- The official image honors environment variables such as `WORDPRESS_DB_*`, `WORDPRESS_DEBUG` and `WORDPRESS_CONFIG_EXTRA` (which is eval'd by `wp-config.php`).
- For additional PHP extensions or customizations, extend the image (`FROM wordpress:<tag>`) and add required configuration.
- Consider Docker Secrets for sensitive values via `_FILE` variants (e.g., `WORDPRESS_DB_PASSWORD_FILE`).

Full upstream docs and supported tags: https://hub.docker.com/_/wordpress
