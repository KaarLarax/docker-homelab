# Nginx Proxy Manager Setup Guide

Nginx Proxy Manager (NPM) is the reverse proxy for the entire stack. It routes external traffic to internal services and manages SSL certificates. This should be the **first service you configure** to avoid domain-related issues later.

## Access the Web UI

```
http://<your-server-ip>:81
```

## Initial Login

Default credentials:
- **Email:** `admin@example.com`
- **Password:** `changeme`

You'll be prompted to change these immediately.

## Domain Strategy

You have two options for accessing your services:

### Option A: Use `.internal` Domains (Local Only)

Simple setup for local network access only. No SSL certificates needed.

```
http://sonarr.internal
http://radarr.internal
http://jellyfin.internal
...
```

**Pros:**
- No domain purchase required
- No SSL configuration
- Works immediately after Pi-hole DNS setup

**Cons:**
- No HTTPS (browsers may show warnings)
- Not accessible from outside your network
- Some features (like Jellyseerr OAuth) may not work

### Option B: Use Your Own Domain (Recommended)

Purchase a domain (e.g., `yourdomain.com`) and use subdomains with Let's Encrypt SSL.

```
https://sonarr.yourdomain.com
https://radarr.yourdomain.com
https://jellyfin.yourdomain.com
...
```

**Pros:**
- HTTPS with valid certificates
- Accessible from anywhere
- All features work correctly
- More professional

**Cons:**
- Requires domain purchase (~$10-15/year)
- Requires DNS configuration
- Slightly more complex setup

## Create Proxy Hosts

### Complete Service Table

| Service | Internal Host | Port | `.internal` Domain | Your Domain |
|---------|---------------|------|-------------------|-------------|
| **Sonarr** | `sonarr` | `8989` | `sonarr.internal` | `sonarr.yourdomain.com` |
| **Radarr** | `radarr` | `7878` | `radarr.internal` | `radarr.yourdomain.com` |
| **Prowlarr** | `prowlarr` | `9696` | `prowlarr.internal` | `prowlarr.yourdomain.com` |
| **Bazarr** | `bazarr` | `6767` | `bazarr.internal` | `bazarr.yourdomain.com` |
| **qBittorrent** | `qbittorrent` | `8080` | `qbittorrent.internal` | `qbittorrent.yourdomain.com` |
| **Jellyfin** | `jellyfin` | `8096` | `jellyfin.yourdomain.com` | `jellyfin.yourdomain.com` |
| **Jellyseerr** | `jellyseerr` | `5055` | `jellyseerr.internal` | `jellyseerr.yourdomain.com` |
| **Bindery** | `bindery` | `8787` | `bindery.internal` | `bindery.yourdomain.com` |
| **Calibre-Web** | `calibre-web` | `8083` | `calibre.internal` | `calibre.yourdomain.com` |
| **Stirling-PDF** | `stirling-pdf` | `8080` | `pdf.internal` | `pdf.yourdomain.com` |
| **Pi-hole** | `pihole` | `8443` | `pihole.internal` | `pihole.yourdomain.com` |

### Example: Create Sonarr Proxy Host

#### For `.internal` Domain (HTTP Only)

1. Go to **Hosts → Proxy Hosts → Add Proxy Host**
2. Fill in:

| Tab | Setting | Value |
|-----|---------|-------|
| **Details** | Domain Names | `sonarr.internal` |
| | Scheme | `http` |
| | Forward Hostname/IP | `sonarr` |
| | Forward Port | `8989` |
| | ☑ Websockets Support | On |
| | ☑ Block Common Exploits | On |
| **SSL** | SSL Certificate | None |

3. Click **Save**

#### For Your Domain (HTTPS with Let's Encrypt)

1. Go to **Hosts → Proxy Hosts → Add Proxy Host**
2. Fill in:

| Tab | Setting | Value |
|-----|---------|-------|
| **Details** | Domain Names | `sonarr.yourdomain.com` |
| | Scheme | `http` |
| | Forward Hostname/IP | `sonarr` |
| | Forward Port | `8989` |
| | ☑ Websockets Support | On |
| | ☑ Block Common Exploits | On |
| **SSL** | SSL Certificate | Request a new SSL Certificate |
| | ☑ Force SSL | On |
| | ☑ HTTP/2 Support | On |
| | ☑ HSTS Enabled | On |
| | ☑ Use a DNS Challenge | On |
| | DNS Provider | Select your provider (Cloudflare, Namecheap, etc.) |

3. Click **Save**

### Repeat for All Services

Create proxy hosts for all services in the table above. Use the same pattern:
- Forward Hostname/IP: Container name (e.g., `sonarr`, `radarr`)
- Forward Port: Service port (e.g., `8989`, `7878`)

## Configure Let's Encrypt SSL (For Your Domain)

### Step 1: Get DNS Provider API Credentials

#### Cloudflare (Recommended)

1. Log into [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. Go to **My Profile → API Tokens**
3. Click **Create Token**
4. Use template: **Edit zone DNS**
5. Permissions:
   - Zone → DNS → Edit
   - Zone → Zone → Read
6. Zone Resources: Include → Specific zone → `yourdomain.com`
7. Click **Continue to summary → Create Token**
8. Copy the token (you won't see it again)

#### Other Providers

- **Namecheap:** Account → API Access → Enable API → Get API Key
- **GoDaddy:** Developer → API Keys → Create new key
- **Google Domains:** Not supported for DNS challenge (use Cloudflare as DNS)

### Step 2: Add DNS Provider to NPM

1. Go to **SSL Certificates → Add SSL Certificate → Let's Encrypt**
2. Fill in:

| Setting | Value |
|---------|-------|
| Domain Names | `*.yourdomain.com` |
| | `yourdomain.com` |
| ☑ Use a DNS Challenge | On |
| DNS Provider | Select your provider |
| Credentials | Paste your API token/key |
| ☑ I Agree to Let's Encrypt Terms | On |

3. Click **Save**

### Step 3: Apply Wildcard Certificate to All Hosts

1. Edit each proxy host
2. Go to **SSL** tab
3. Select your wildcard certificate from the dropdown
4. ☑ Force SSL
5. ☑ HTTP/2 Support
6. ☑ HSTS Enabled
7. Click **Save**

## Configure Trusted Proxy Headers

Some services need to know the real client IP when behind a reverse proxy. Add this custom Nginx configuration to services that need it.

### Services That Need Trusted Proxy Headers

| Service | Why |
|---------|-----|
| **Jellyfin** | Show correct client IP in logs |
| **qBittorrent** | Bypass authentication for local network |
| **Jellyseerr** | OAuth authentication |

### Add Custom Headers in NPM

1. Edit the proxy host (e.g., Jellyfin)
2. Go to **Advanced** tab
3. Paste this configuration:

```nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

4. Click **Save**

### qBittorrent: Bypass Authentication for NPM

To allow qBittorrent to accept requests from NPM without authentication:

1. Edit qBittorrent proxy host in NPM
2. Go to **Advanced** tab
3. Add:

```nginx
proxy_set_header X-Forwarded-For $remote_addr;
```

4. In qBittorrent Web UI:
   - Go to **Tools → Options → Web UI**
   - ☑ **Bypass authentication for clients on localhost**
   - ☑ **Bypass authentication for clients in whitelisted subnets**
   - Add: `172.20.0.0/24` (your Docker network)

## Configure Access Lists (Optional)

To restrict access to certain services:

1. Go to **Access Lists → Add Access List**
2. Add allowed IPs or require authentication
3. Apply the access list to proxy hosts

### Basic Authentication

1. Go to **Access Lists → Add Access List**
2. **Authorization** tab:
   - Add user and password
3. Apply to proxy hosts that need protection

## Configure Redirect Hosts (Optional)

To redirect old domains or URLs:

1. Go to **Hosts → Redirect Hosts → Add Redirect Host**
2. Fill in:
   - Domain Names: `old.yourdomain.com`
   - Scheme: `https`
   - Target URL: `https://new.yourdomain.com`
   - Status Code: `301` (permanent) or `302` (temporary)

## Configure Streams (Optional)

For non-HTTP traffic (e.g., TCP/UDP):

1. Go to **Hosts → Streams → Add Stream**
2. Fill in:
   - Listening Port: `25565` (example)
   - Forwarding Host: `minecraft-server`
   - Forwarding Port: `25565`
   - Protocol: `TCP`

## Next Steps

→ [Set up Pi-hole](02-pi-hole.md) to configure DNS for your `.internal` domains

## Reference

- [Nginx Proxy Manager GitHub](https://github.com/NginxProxyManager/nginx-proxy-manager)
- [Nginx Proxy Manager Documentation](https://nginxproxymanager.com/setup/)
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)
