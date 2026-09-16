# Configuration Guides

Step-by-step guides for configuring each service in this stack. Follow them in order for the best experience.

## Setup Order

The guides are numbered by dependency. Start with infrastructure (NPM + Pi-hole), then work through the services:

| # | Service | Guide | Description |
|---|---------|-------|-------------|
| 01 | **Nginx Proxy Manager** | [Setup Guide](01-nginx-proxy-manager.md) | Reverse proxy -- configure first for domains |
| 02 | **Pi-hole** | [Setup Guide](02-pi-hole.md) | DNS -- configure `.internal` domains |
| 03 | **qBittorrent** | [Setup Guide](03-qbittorrent.md) | Download client -- base of the stack |
| 04 | **FlareSolverr** | [Setup Guide](04-flaresolverr.md) | Cloudflare bypass -- required before Prowlarr |
| 05 | **Prowlarr** | [Setup Guide](05-prowlarr.md) | Indexer manager -- connects to qBittorrent |
| 06 | **Radarr** | [Setup Guide](06-radarr.md) | Movie automation -- connects to Prowlarr + qBittorrent |
| 07 | **Sonarr** | [Setup Guide](07-sonarr.md) | TV automation -- connects to Prowlarr + qBittorrent |
| 08 | **Bazarr** | [Setup Guide](08-bazarr.md) | Subtitles -- connects to Radarr + Sonarr |
| 09 | **Jellyfin** | [Setup Guide](09-jellyfin.md) | Media server -- streams your library |
| 10 | **Jellyseerr** | [Setup Guide](10-jellyseerr.md) | Request management -- connects to Jellyfin + Radarr + Sonarr |
| 11 | **Bindery** | [Setup Guide](11-bindery.md) | Book download automation -- connects to qBittorrent |
| 12 | **Calibre-Web** | [Setup Guide](12-calibre-web.md) | Ebook library manager |
| 13 | **Adding New Containers** | [Guide](13-adding-new-containers.md) | How to add new services to the stack |

## Connection Map

```
Nginx Proxy Manager (reverse proxy) ← Configure first!
        ↓
Pi-hole (DNS for .internal domains)
        ↓
qBittorrent
        ↓
FlareSolverr ← Prowlarr → Radarr
                  ↓          ↓
               Sonarr     Bazarr ← Radarr + Sonarr
                  ↓
              Jellyfin ← Jellyseerr
                  ↓
            Bindery → Calibre-Web (books pipeline)
```

## Domain Reference

| Service | `.internal` Domain | Your Domain |
|---------|-------------------|-------------|
| Sonarr | `sonarr.internal` | `sonarr.yourdomain.com` |
| Radarr | `radarr.internal` | `radarr.yourdomain.com` |
| Prowlarr | `prowlarr.internal` | `prowlarr.yourdomain.com` |
| Bazarr | `bazarr.internal` | `bazarr.yourdomain.com` |
| qBittorrent | `qbittorrent.internal` | `qbittorrent.yourdomain.com` |
| Jellyfin | `jellyfin.internal` | `jellyfin.yourdomain.com` |
| Jellyseerr | `jellyseerr.internal` | `jellyseerr.yourdomain.com` |
| Bindery | `bindery.internal` | `bindery.yourdomain.com` |
| Calibre-Web | `calibre.internal` | `calibre.yourdomain.com` |
| Stirling-PDF | `pdf.internal` | `pdf.yourdomain.com` |
| Pi-hole | `pihole.internal` | `pihole.yourdomain.com` |

## Quick Reference: API Keys

You'll need to exchange API keys between services. Here's where to find them:

| Service | Location |
|---------|----------|
| **Radarr** | Settings → General → API Key |
| **Sonarr** | Settings → General → API Key |
| **Prowlarr** | Settings → General → API Key |
| **Bazarr** | Settings → General → API Key |
| **qBittorrent** | Tools → Options → Web UI → Show API Key |

## Internal Docker URLs

When connecting services to each other, use these internal URLs (not localhost or external IPs):

| Service | Internal URL |
|---------|-------------|
| qBittorrent | `http://qbittorrent:8080` |
| Radarr | `http://radarr:7878` |
| Sonarr | `http://sonarr:8989` |
| Prowlarr | `http://prowlarr:9696` |
| Bazarr | `http://bazarr:6767` |
| Jellyfin | `http://jellyfin:8096` |
| Jellyseerr | `http://jellyseerr:5055` |
| Bindery | `http://bindery:8787` |
| Calibre-Web | `http://calibre-web:8083` |
| FlareSolverr | `http://flaresolverr:8191` |
