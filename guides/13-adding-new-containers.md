# Adding New Containers Guide

This guide explains how to add a new container to your Docker stack while maintaining consistency with the existing setup.

## Directory Structure

### Option A: Part of Media Stack

If the new container is related to media/books management, place it inside `media-stack/`:

```
docker/
├── compose.yaml                      # Root orchestrator
├── media-stack/
│   ├── compose.yaml                  # Main media services
│   ├── .env                          # Shared environment variables
│   ├── bindery/
│   │   └── compose.yaml
│   ├── calibre-web/
│   │   └── compose.yaml
│   └── your-new-service/             # NEW
│       └── compose.yaml
```

### Option B: Independent Service

If the container is unrelated to media (e.g., monitoring, home automation), place it in the root:

```
docker/
├── compose.yaml                      # Root orchestrator
├── media-stack/
├── nginx-proxy-manager/
├── pihole/
├── pdfutils/
└── your-new-service/                 # NEW
    └── compose.yaml
```

## Step-by-Step: Adding a New Container

### Step 1: Create the Directory Structure

```bash
# For media-related services
mkdir -p media-stack/your-new-service

# OR for independent services
mkdir -p your-new-service
```

### Step 2: Create the compose.yaml

Use the standard structure (same order as other services):

```yaml
services:
  your-new-service:
    image: maintainer/your-new-service:latest
    container_name: your-new-service
    restart: unless-stopped
    env_file:
      - ../.env  # If inside media-stack/, otherwise - .env
    networks:
      - proxy
    # ports:
    #   - "PORT:PORT"  # Only if accessing directly (not through NPM)
    environment:
      TZ: ${TZ}
      # Other environment variables
    volumes:
      - "./config:/config"  # Relative path from compose.yaml location
      # Add other volumes as needed

networks:
  proxy:
    external: true
```

**Key Points:**
- Use `env_file: - ../.env` if inside `media-stack/` to share environment variables
- Use `env_file: - .env` if in root directory (create your own `.env` if needed)
- Always use relative paths for volumes (`./config` not `/absolute/path`)
- Include `networks: - proxy` to connect to the shared network
- Comment out `ports` if accessing only through NPM

### Step 3: Add to Root compose.yaml

Edit the root `compose.yaml` and add your new service:

```yaml
include:
  - nginx-proxy-manager/compose.yaml
  - media-stack/compose.yaml
  - media-stack/bindery/compose.yaml
  - media-stack/calibre-web/compose.yaml
  - media-stack/your-new-service/compose.yaml  # ADD THIS LINE
  - pihole/compose.yaml
  - pdfutils/compose.yaml
```

### Step 4: Test the Configuration

```bash
# Validate the configuration
docker compose config

# Start the new service
docker compose up -d your-new-service

# Check logs
docker compose logs -f your-new-service
```

### Step 5: Configure Nginx Proxy Manager

1. Access NPM at `http://<server-ip>:81`
2. Go to **Hosts → Proxy Hosts → Add Proxy Host**
3. Fill in:

| Setting | Value |
|---------|-------|
| Domain Names | `your-service.internal` |
| Scheme | `http` |
| Forward Hostname/IP | `your-new-service` |
| Forward Port | `PORT` |
| ☑ Websockets Support | On (if needed) |
| ☑ Block Common Exploits | On |

4. Click **Save**

**For external access with your domain:**

| Setting | Value |
|---------|-------|
| Domain Names | `your-service.yourdomain.com` |
| SSL Certificate | Request new or use wildcard |
| ☑ Force SSL | On |

### Step 6: Configure Pi-hole Local DNS

1. Access Pi-hole at `https://<server-ip>:8443/admin`
2. Go to **Local DNS → DNS Records**
3. Add:

| Domain | IP Address |
|--------|------------|
| `your-service.internal` | `192.168.1.100` (your server's IP) |

4. Click **Add**

### Step 7: Update Documentation

Update the following files to keep documentation consistent:

#### 1. guides/README.md

Add to the setup order table:

```markdown
| 13 | **Your New Service** | [Setup Guide](13-your-new-service.md) | Description |
```

Add to the domain reference table:

```markdown
| Your New Service | `your-service.internal` | `your-service.yourdomain.com` |
```

Add to internal Docker URLs:

```markdown
| Your New Service | `http://your-new-service:PORT` |
```

#### 2. README.md (root)

Update the project structure:

```markdown
├── media-stack/
│   ├── your-new-service/
│   │   └── compose.yaml              # Your new service
```

Update the services table:

```markdown
| **Your New Service** | Description | `PORT` | [maintainer/repo](https://github.com/...) |
```

#### 3. Create a Setup Guide

Create `guides/13-your-new-service.md` following the same structure as other guides:

- Access the Web UI (direct IP + NPM domain)
- Initial configuration
- Connect to other services (if needed)
- Next Steps link

### Step 8: Verify Everything Works

```bash
# Check all services are running
docker compose ps

# Test DNS resolution
ping your-service.internal

# Test NPM routing
curl http://your-service.internal

# Check service logs
docker compose logs your-new-service
```

## Example: Adding Home Assistant

Here's a complete example of adding Home Assistant:

### 1. Directory Structure

```bash
mkdir -p homeassistant
```

### 2. compose.yaml

```yaml
services:
  homeassistant:
    image: ghcr.io/home-assistant/home-assistant:stable
    container_name: homeassistant
    restart: unless-stopped
    networks:
      - proxy
    # ports:
    #   - "8123:8123"
    environment:
      TZ: ${TZ}
    volumes:
      - "./config:/config"
      - /run/dbus:/run/dbus:ro

networks:
  proxy:
    external: true
```

### 3. Root compose.yaml

```yaml
include:
  - nginx-proxy-manager/compose.yaml
  - media-stack/compose.yaml
  - media-stack/bindery/compose.yaml
  - media-stack/calibre-web/compose.yaml
  - pihole/compose.yaml
  - pdfutils/compose.yaml
  - homeassistant/compose.yaml  # ADDED
```

### 4. NPM Configuration

| Setting | Value |
|---------|-------|
| Domain Names | `homeassistant.internal` |
| Forward Hostname/IP | `homeassistant` |
| Forward Port | `8123` |
| ☑ Websockets Support | On |

### 5. Pi-hole DNS

| Domain | IP Address |
|--------|------------|
| `homeassistant.internal` | `192.168.1.100` |

### 6. Documentation Updates

Update `guides/README.md`, `README.md`, and create `guides/13-homeassistant.md`.

## Checklist: Adding a New Container

- [ ] Created directory (`media-stack/your-service/` or `your-service/`)
- [ ] Created `compose.yaml` with standard structure
- [ ] Used relative paths for volumes
- [ ] Connected to `proxy` network
- [ ] Added to root `compose.yaml` in `include` section
- [ ] Tested with `docker compose config`
- [ ] Started service with `docker compose up -d your-service`
- [ ] Configured NPM proxy host
- [ ] Added Pi-hole DNS record for `.internal` domain
- [ ] Updated `guides/README.md` (setup order, domain table, URLs)
- [ ] Updated root `README.md` (project structure, services table)
- [ ] Created setup guide `guides/XX-your-service.md`
- [ ] Verified DNS resolution (`ping your-service.internal`)
- [ ] Verified NPM routing (`curl http://your-service.internal`)

## Common Patterns

### Service That Needs qBittorrent

```yaml
environment:
  QBITTORRENT_HOST: qbittorrent
  QBITTORRENT_PORT: 8080
```

### Service That Needs *arr Apps

```yaml
environment:
  RADARR_URL: http://radarr:7878
  SONARR_URL: http://sonarr:8989
  RADARR_API_KEY: ${RADARR_API_KEY}
```

### Service with Database

```yaml
services:
  your-service:
    # ... your service config ...
  
  your-service-db:
    image: postgres:15
    container_name: your-service-db
    restart: unless-stopped
    networks:
      - proxy
    environment:
      POSTGRES_DB: your-service
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - "./db-data:/var/lib/postgresql/data"

networks:
  proxy:
    external: true
```

## Troubleshooting

### Service Can't Connect to Others

Ensure the service is on the `proxy` network:

```bash
docker network inspect proxy | grep your-service
```

### NPM Returns 502 Bad Gateway

Check that:
1. The container is running: `docker compose ps your-service`
2. The port is correct in NPM configuration
3. The container name matches the forward hostname

### DNS Not Resolving

1. Check Pi-hole DNS records
2. Flush DNS cache on client: `sudo systemd-resolve --flush-caches`
3. Test from server: `dig your-service.internal @127.0.0.1`

## Next Steps

→ [Back to README](../README.md)
