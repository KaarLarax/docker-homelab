# Docker Homelab

A fully orchestrated Docker Compose setup for a self-hosted media server, DNS filtering, reverse proxy, and utility tools. Everything runs behind a single Docker network with centralized management.

## Architecture

```
                          ┌─────────────────────────────┐
                          │    Nginx Proxy Manager       │
                          │   :80  :81  :443             │
                          └──────────┬──────────────────┘
                                     │  proxy network (bridge)
        ┌────────────────────────────┼────────────────────────────┐
        │                            │                            │
  ┌─────┴──────┐            ┌───────┴────────┐           ┌──────┴───────┐
  │ Media Stack │            │  Books Stack   │           │   Pi-hole    │
  │             │            │                │           │  DNS :53     │
  │ Sonarr      │            │  Bindery       │           │  Admin :8443 │
  │ Radarr      │            │  Calibre-Web   │           └──────────────┘
  │ Prowlarr    │            └────────────────┘
  │ Bazarr      │
  │ qBittorrent │            ┌────────────────┐
  │ Jellyfin    │            │    PDF Utils   │
  │ Jellyseerr  │            │  Stirling-PDF  │
  │ FlareSolverr│            └────────────────┘
  └─────────────┘
```

All containers share a single external Docker bridge network (`proxy`), enabling internal DNS resolution between services and centralized reverse proxy routing through Nginx Proxy Manager.

## Prerequisites

- [Docker](https://docs.docker.com/engine/install/) (v24+)
- [Docker Compose](https://docs.docker.com/compose/install/) v2 (with `include` support)
- A dedicated Docker network named `proxy`

### Create the proxy network (first-time setup)

```bash
docker network create --driver bridge --subnet 172.20.0.0/24 --gateway 172.20.0.1 proxy
```

## Quick Start

```bash
git clone git@github.com:KaarLarax/docker-homelab.git
cd docker-homelab

# Configure environment variables
cp media-stack/.env.example media-stack/.env
cp pihole/.env.example pihole/.env

# Edit .env files with your own values
nano media-stack/.env
nano pihole/.env

# Start everything
docker compose up -d
```

That's it. All services start from the root directory with a single command.

## Project Structure

```
.
├── compose.yaml                      # Root orchestrator (includes all stacks)
├── guides/                           # Step-by-step setup guides
│   ├── README.md                     # Guides overview & connection map
│   ├── 01-nginx-proxy-manager.md     # Reverse proxy setup (configure first)
│   ├── 02-pi-hole.md                 # DNS & .internal domains setup
│   ├── 03-qbittorrent.md             # Download client setup
│   ├── 04-flaresolverr.md            # Cloudflare bypass (before Prowlarr)
│   ├── 05-prowlarr.md                # Indexer manager setup
│   ├── 06-radarr.md                  # Movie automation setup
│   ├── 07-sonarr.md                  # TV automation setup
│   ├── 08-bazarr.md                  # Subtitle management setup
│   ├── 09-jellyfin.md                # Media server setup
│   ├── 10-jellyseerr.md              # Request management setup
│   ├── 11-bindery.md                 # Book automation setup
│   ├── 12-calibre-web.md             # Ebook library setup
│   └── 13-adding-new-containers.md   # How to add new services
├── media-stack/
│   ├── compose.yaml                  # Sonarr, Radarr, Prowlarr, Bazarr, qBittorrent, Jellyfin, Jellyseerr, FlareSolverr
│   ├── .env                          # Shared environment variables (TZ, PUID, PGID)
│   ├── .env.example
│   ├── bindery/
│   │   └── compose.yaml              # Book management (audiobooks & ebooks)
│   └── calibre-web/
│       └── compose.yaml              # Calibre-Web Automated (ebook reader & manager)
├── nginx-proxy-manager/
│   └── compose.yaml                  # Reverse proxy with Let's Encrypt support
├── pihole/
│   ├── compose.yaml                  # Network-wide ad blocking & DNS
│   ├── .env
│   └── .env.example
└── pdfutils/
    └── compose.yaml                  # Stirling-PDF (PDF manipulation tools)
```

## Services

### Media Stack

| Service | Description | Default Port | Docs |
|---------|-------------|:------------:|------|
| **Sonarr** | TV series management & automation | `8989` | [linuxserver/sonarr](https://github.com/linuxserver/docker-sonarr) |
| **Radarr** | Movie management & automation | `7878` | [linuxserver/radarr](https://github.com/linuxserver/docker-radarr) |
| **Prowlarr** | Indexer manager for *arr apps | `9696` | [linuxserver/prowlarr](https://github.com/linuxserver/docker-prowlarr) |
| **Bazarr** | Subtitle management | `6767` | [linuxserver/bazarr](https://github.com/linuxserver/docker-bazarr) |
| **qBittorrent** | Torrent client | `8080` / `6881` | [linuxserver/qbittorrent](https://github.com/linuxserver/docker-qbittorrent) |
| **Jellyfin** | Media server (streaming) | `8096` | [linuxserver/jellyfin](https://github.com/linuxserver/docker-jellyfin) |
| **Jellyseerr** | Request management for Jellyfin | `5055` | [fallenbagel/jellyseerr](https://github.com/Fallenbagel/jellyseerr) |
| **FlareSolverr** | Cloudflare bypass for indexers | `8191` | [FlareSolverr](https://github.com/FlareSolverr/FlareSolverr) |

### Books

| Service | Description | Default Port | Docs |
|---------|-------------|:------------:|------|
| **Bindery** | Audiobook & ebook download automation | `8787` | [vavallee/bindery](https://github.com/vavallee/bindery) |
| **Calibre-Web Automated** | Ebook library & reader | `8083` | [crocodilestick/calibre-web-automated](https://github.com/crocodilestick/Calibre-Web-Automated) |

### Infrastructure

| Service | Description | Default Port | Docs |
|---------|-------------|:------------:|------|
| **Nginx Proxy Manager** | Reverse proxy with SSL | `80` / `81` / `443` | [jc21/nginx-proxy-manager](https://github.com/NginxProxyManager/nginx-proxy-manager) |
| **Pi-hole** | Network-wide DNS ad blocking | `53` / `8443` | [pi-hole/docker-pi-hole](https://github.com/pi-hole/docker-pi-hole) |

### Utilities

| Service | Description | Default Port | Docs |
|---------|-------------|:------------:|------|
| **Stirling-PDF** | PDF manipulation toolkit | `8080` | [Stirling-Tools/Stirling-PDF](https://github.com/Stirling-Tools/Stirling-PDF) |

## Configuration Guides

Step-by-step setup guides are available in the [`guides/`](guides/) directory. Follow them in order:

1. [Nginx Proxy Manager](guides/01-nginx-proxy-manager.md) - Reverse proxy (configure first)
2. [Pi-hole](guides/02-pi-hole.md) - DNS & `.internal` domains
3. [qBittorrent](guides/03-qbittorrent.md) - Download client
4. [FlareSolverr](guides/04-flaresolverr.md) - Cloudflare bypass (required before Prowlarr)
5. [Prowlarr](guides/05-prowlarr.md) - Indexer manager
6. [Radarr](guides/06-radarr.md) - Movie automation
7. [Sonarr](guides/07-sonarr.md) - TV automation
8. [Bazarr](guides/08-bazarr.md) - Subtitle management
9. [Jellyfin](guides/09-jellyfin.md) - Media server
10. [Jellyseerr](guides/10-jellyseerr.md) - Request management
11. [Bindery](guides/11-bindery.md) - Book automation
12. [Calibre-Web](guides/12-calibre-web.md) - Ebook library
13. [Adding New Containers](guides/13-adding-new-containers.md) - How to add new services

See the [guides README](guides/README.md) for a complete overview with connection maps, domain reference, and API key locations.

## Recommended External Resources

For optimal configuration of the *arr stack and downloaders, follow [Trash Guides](https://trash-guides.info/):

| Service | Trash Guide |
|---------|-------------|
| **Radarr** | [Radarr Guide](https://trash-guides.info/Radarr/) |
| **Sonarr** | [Sonarr Guide](https://trash-guides.info/Sonarr/) |
| **Prowlarr** | [Prowlarr Guide](https://trash-guides.info/Prowlarr/) |
| **Bazarr** | [Bazarr Guide](https://trash-guides.info/Bazarr/) |
| **qBittorrent** | [qBittorrent Guide](https://trash-guides.info/Downloaders/qBittorrent/) |

These guides cover quality profiles, naming conventions, media management, and download client setup.

## Configuration

### Environment Variables

#### `media-stack/.env`

| Variable | Description | Example |
|----------|-------------|---------|
| `TZ` | Timezone for all containers | `America/Mexico_City` |
| `PUID` | User ID for file permissions | `1000` |
| `PGID` | Group ID for file permissions | `1000` |

Find your PUID/PGID:
```bash
id -u   # PUID
id -g   # PGID
```

#### `pihole/.env`

| Variable | Description | Example |
|----------|-------------|---------|
| `TZ` | Timezone | `America/Mexico_City` |
| `INITIAL_PASSWORD` | Pi-hole admin password | `changeme` |

### Volume Structure

All data is stored relative to each compose file's directory:

```
media-stack/
├── bazarr/config/        # Bazarr configuration
├── jellyfin/config/      # Jellyfin configuration
├── jellyseerr/config/    # Jellyseerr configuration
├── prowlarr/config/      # Prowlarr configuration
├── qbittorrent/config/   # qBittorrent configuration
├── radarr/config/        # Radarr configuration
├── sonarr/config/        # Sonarr configuration
├── bindery/config/       # Bindery configuration
├── calibre-web/config/   # Calibre-Web configuration
└── data/
    ├── media/            # Your organized media library
    │   ├── movies/       # Movies (managed by Radarr)
    │   ├── tv/           # TV shows (managed by Sonarr)
    │   └── books/        # Books collection
    │       ├── books/           # Ebooks (managed by Calibre-Web)
    │       ├── audiobooks/      # Audiobooks (managed by Bindery)
    │       └── books-ingest/    # Ingest folder for Calibre-Web automation
    └── torrents/         # Downloaded torrents from qBittorrent
        ├── movies/       # Movie downloads
        ├── tv/           # TV show downloads
        └── books/        # Book downloads
            ├── books/           # Ebook downloads (for Bindery)
            └── audiobooks/      # Audiobook downloads (for Bindery)
```

**Important:** The `torrents/` and `media/` directories must be separate to allow *arr apps to hardlink or move files efficiently. qBittorrent downloads to `torrents/`, then Sonarr/Radarr/calibre-web organize and move files to `media/`.

### GPU Acceleration (Jellyfin)

Jellyfin is configured to use `/dev/dri` for hardware transcoding. If you have a different GPU, update the `devices` section in `media-stack/compose.yaml`:

```yaml
devices:
  - /dev/dri:/dev/dri          # Intel/AMD
  # - /dev/nvidia0:/dev/nvidia0  # NVIDIA (also uncomment NVIDIA env vars)
```

## How It All Works Together

### Networking

All containers connect to a single external Docker bridge network called `proxy`. This network is created once and persists across restarts:

```bash
docker network create --driver bridge --subnet 172.20.0.0/24 --gateway 172.20.0.1 proxy
```

This enables:
- **Internal DNS resolution**: Containers can reach each other by service name (e.g., Sonarr can reach Jellyfin at `http://jellyfin:8096`)
- **Centralized reverse proxy**: Nginx Proxy Manager routes external traffic to the correct container
- **Isolation**: Services are not directly exposed to the host network unless explicitly mapped

### Reverse Proxy Flow

```
Client Request → Nginx Proxy Manager (:80/:443)
                      │
                      ├─→ sonarr:8989      (via internal DNS)
                      ├─→ radarr:7878
                      ├─→ jellyfin:8096
                      ├─→ jellyseerr:5055
                      └─→ ...
```

Configure proxy hosts in the Nginx Proxy Manager admin panel at `:81`.

### DNS & Ad Blocking

Pi-hole listens on port `53` (DNS) and `8443` (admin panel). Point your router's DNS to this server's IP to enable network-wide ad blocking.

### Data Flow (Media Stack)

```
qBittorrent downloads → data/torrents/
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
          Sonarr          Radarr          Bindery
          (TV)           (Movies)        (Books)
              │               │               │
              ▼               ▼               ▼
         Renamed & organized into data/media/
              │
              ▼
          Jellyfin (streams to clients)
```

## Common Commands

```bash
# Start all services
docker compose up -d

# Stop all services
docker compose down

# View logs (all services)
docker compose logs -f

# View logs (specific service)
docker compose logs -f sonarr

# Restart a single service
docker compose restart radarr

# Update all images
docker compose pull && docker compose up -d

# Check service health
docker compose ps
```

## Troubleshooting

### "network proxy not found"
Create the network first:
```bash
docker network create --driver bridge --subnet 172.20.0.0/24 --gateway 172.20.0.1 proxy
```

### Permission errors on volumes
Ensure `PUID` and `PGID` in `.env` match your host user:
```bash
id -u && id -g
```

### Port conflicts
- Port `53` (Pi-hole) may conflict with `systemd-resolved`. Disable it:
  ```bash
  sudo systemctl stop systemd-resolved
  sudo systemctl disable systemd-resolved
  ```
- Port `80`/`443` must be free for Nginx Proxy Manager.

### Services can't reach each other
Verify all containers are on the `proxy` network:
```bash
docker network inspect proxy
```

## License

This project is for personal use. Each service retains its own license -- see the linked repositories above.
