# Prowlarr Setup Guide

Prowlarr is the indexer manager. It connects to your indexers and syncs them automatically to Radarr, Sonarr, and other *arr apps.

## Access the Web UI

### Via Direct IP (Initial Setup)

```
http://<your-server-ip>:9696
```

### Via NPM Domain (After NPM Setup)

```
http://prowlarr.internal
# or
https://prowlarr.yourdomain.com
```

## Initial Setup Wizard

On first launch, Prowlarr will show a setup wizard:

1. **Authentication** → Set a username and password (or skip for local-only access)
2. **Update Mechanism** → Choose `External` (Docker handles updates)

## Add Indexers

### 1. Go to **Indexers → Add Indexer**

Search for your preferred indexers. Common ones:

| Type | Examples |
|------|----------|
| **Public** | 1337x, The Pirate Bay, YTS, TorrentGalaxy |
| **Semi-Private** | TorrentLeech, IPTorrents |
| **Private** | PTP, BLU, HDB, RED, OPS |

### 2. Configure Each Indexer

For each indexer:
- Select it from the list
- Fill in credentials if required (username/password/API key)
- Set the **Category** mapping (usually auto-detected)
- Click **Test** to verify it works
- Click **Save**

### 3. Add FlareSolverr (for Cloudflare-protected indexers)

If you have indexers protected by Cloudflare:

1. Go to **Settings → Apps → Add FlareSolverr**
2. Enter the URL:
   ```
   http://flaresolverr:8191
   ```
3. Click **Test** → **Save**

## Limit Connections (Important for DNS Health)

To avoid saturating your local DNS (Pi-hole or router), limit the number of concurrent indexer requests.

Go to **Settings → Apps → Prowlarr** (or **Settings → General** depending on version):

| Setting | Recommended Value | Why |
|---------|-------------------|-----|
| Maximum concurrent client connections | `2` | Limits simultaneous requests to indexers |
| Maximum grabber connections | `2` | Limits simultaneous download client connections |

> **Why this matters:** When Radarr/Sonarr trigger searches across many indexers simultaneously, each indexer query generates DNS lookups. With Pi-hole as your DNS, too many concurrent requests can cause timeouts or overload the DNS service. These limits keep searches manageable.

### Per-Indexer Rate Limiting

For each indexer, you can also set a specific rate limit:

1. Edit an indexer
2. Go to the **Advanced** section
3. Set:

| Setting | Recommended Value |
|---------|-------------------|
| Priority | `25` (default) |
| Grab URL | Leave as default |

> **Tip:** If you notice DNS timeouts during searches, reduce the number of active indexers or increase the interval between searches in Radarr/Sonarr.

## Connect Download Client (qBittorrent)

1. Go to **Settings → Download Clients → Add Download Client**
2. Select **qBittorrent**
3. Fill in:

| Setting | Value |
|---------|-------|
| Name | `qBittorrent` |
| Host | `qbittorrent` |
| Port | `8080` |
| Username | `admin` (or your custom user) |
| Password | Your qBittorrent password |
| Category | Leave empty (managed by *arr apps) |

4. Click **Test** → **Save**

## Connect *arr Applications

This is the key feature -- Prowlarr syncs all your indexers to Radarr and Sonarr automatically.

### Get Prowlarr API Key

1. Go to **Settings → General**
2. Copy the **API Key**

### Add Radarr to Prowlarr

1. Go to **Settings → Apps → Add Application**
2. Select **Radarr**
3. Fill in:

| Setting | Value |
|---------|-------|
| Name | `Radarr` |
| Prowlarr Server | `http://prowlarr:9696` |
| Radarr Server | `http://radarr:7878` |
| API Key | *(paste Radarr's API key)* |
| Sync Level | `Full Sync` |

4. Click **Test** → **Save**

### Add Sonarr to Prowlarr

1. Go to **Settings → Apps → Add Application**
2. Select **Sonarr**
3. Fill in:

| Setting | Value |
|---------|-------|
| Name | `Sonarr` |
| Prowlarr Server | `http://prowlarr:9696` |
| Sonarr Server | `http://sonarr:8989` |
| API Key | *(paste Sonarr's API key)* |
| Sync Level | `Full Sync` |

4. Click **Test** → **Save**

### How to Get Radarr/Sonarr API Keys

- **Radarr:** Go to `http://radarr:7878` → **Settings → General → API Key**
- **Sonarr:** Go to `http://sonarr:8989` → **Settings → General → API Key**

## Verify Sync

After adding Radarr and Sonarr:

1. Go to **Indexers** in Prowlarr
2. You should see all indexers listed
3. Each indexer should have icons showing which apps it's synced to (Radarr, Sonarr)
4. Check Radarr/Sonarr → **Settings → Indexers** to confirm they received the indexers

## Next Steps

→ [Set up Radarr](06-radarr.md)
→ [Set up Sonarr](07-sonarr.md)

## Reference

- [Trash Guides: Prowlarr](https://trash-guides.info/Prowlarr/)
