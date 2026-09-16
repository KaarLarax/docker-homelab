# qBittorrent Setup Guide

qBittorrent is the download client for the entire stack. All *arr apps will send torrents here.

## Access the Web UI

### Via Direct IP (Initial Setup)

```
http://<your-server-ip>:8080
```

### Via NPM Domain (After NPM Setup)

```
http://qbittorrent.internal
# or
https://qbittorrent.yourdomain.com
```

Default credentials:
- **Username:** `admin`
- **Password:** `adminadmin` (change this immediately)

## Initial Configuration

### 1. Change Default Password

Go to **Tools → Options → Web UI** and change the default password.

### 2. Disable Strict Validation (Required for NPM Access)

When accessing qBittorrent through NPM (via domain), you need to disable strict validation to avoid authentication errors.

Go to **Tools → Options → Web UI → Authentication**:

| Setting | Value |
|---------|-------|
| ☑ **Bypass authentication for clients on localhost** | On |
| ☑ **Bypass authentication for clients in whitelisted subnets** | On |
| Whitelisted subnets | `172.20.0.0/24` (your Docker proxy network) |

Then, in the NPM proxy host for qBittorrent, add this custom configuration in the **Advanced** tab:

```nginx
proxy_set_header X-Forwarded-For $remote_addr;
```

This tells qBittorrent that requests are coming from the trusted proxy network, not from an external source.

> **Why this is needed:** qBittorrent has CSRF protection that blocks requests from domains it doesn't recognize. By whitelisting the Docker subnet and forwarding the correct headers, NPM can proxy requests without triggering security blocks.

### 3. Set Up Categories

Go to **Tools → Options → BitTorrent → Torrent Management** and enable:

- ☑ **Use Subcategories**

Then manually add these categories (right-click in the categories panel → **Add Category**):

| Category | Save Path |
|----------|-----------|
| `movies` | `/data/torrents/movies` |
| `tv` | `/data/torrents/tv` |
| `books` | `/data/torrents/books/books` |
| `audiobooks` | `/data/torrents/books/audiobooks` |

### 4. Configure Connection

Go to **Tools → Options → Connection**:

| Setting | Value |
|---------|-------|
| Listening Port | `6881` |
| ☑ Use UPnP / NAT-PMP | Off |
| ☑ Enable DHT | On |
| ☑ Enable Peer Exchange (PeX) | On |
| ☑ Enable Local Peer Discovery | On |

### 5. Configure Speed

Go to **Tools → Options → Speed**:

| Setting | Value |
|---------|-------|
| Global Rate Limits | Set according to your connection |
| ☑ Apply rate limit to TCP | On |
| ☑ Apply rate limit to uTP | On |

### 6. Limit Connections (Important for DNS Health)

To avoid saturating your local DNS (Pi-hole or router), limit the number of concurrent connections.

Go to **Tools → Options → Connection**:

| Setting | Recommended Value | Why |
|---------|-------------------|-----|
| Global maximum number of connections | `500` | Prevents overwhelming your router/DNS |
| Maximum number of connections per torrent | `100` | Limits per-torrent load |
| Global maximum number of upload slots | `50` | Prevents upload saturation |
| Maximum number of upload slots per torrent | `10` | Fair distribution |

> **Why this matters:** Each peer connection can trigger DNS lookups. With hundreds of peers and limited DNS capacity (especially Pi-hole), you can cause DNS timeouts or crashes. These limits keep your network stable.

### 7. Limit DNS Lookup Rate

Go to **Tools → Options → Advanced**:

| Setting | Recommended Value |
|---------|-------------------|
| ☑ Enable embedded tracker | Off |
| Network interface | Select your network interface |
| IP Address | Leave blank |

> **Note:** qBittorrent doesn't have a direct "DNS lookup rate limit" setting, but limiting connections (step 6) effectively reduces DNS load.

### 8. Enable Auto-Add Torrents (Optional)

Go to **Tools → Options → Downloads**:

- ☑ **Enable automatic .torrent file adding**
- Set watch folder if needed

## Connect to Prowlarr

After setting up qBittorrent, you'll need its API key to connect it to Prowlarr:

1. Go to **Tools → Options → Web UI**
2. Click **Show API Key** (or find it in the URL bar when logged in)
3. Copy the API key -- you'll need it in the [Prowlarr setup](05-prowlarr.md)

## Next Steps

→ [Set up FlareSolverr](04-flaresolverr.md) (required before Prowlarr)

## Reference

- [Trash Guides: qBittorrent](https://trash-guides.info/Downloaders/qBittorrent/)
