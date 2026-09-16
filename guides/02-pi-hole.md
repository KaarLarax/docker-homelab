# Pi-hole Setup Guide

Pi-hole is a network-wide ad blocker and DNS server. It blocks ads, trackers, and malicious domains for all devices on your network.

## Access the Web UI

```
https://<your-server-ip>:8443/admin
```

> Note: Pi-hole uses HTTPS on port 8443 in this setup.

## Initial Login

The admin password is set in `pihole/.env`:

```env
INITIAL_PASSWORD=your_password_here
```

Login with:
- **Password:** The value from `INITIAL_PASSWORD`

## Configure DNS

### 1. Upstream DNS Servers

Go to **Settings → DNS**:

#### Recommended Upstream DNS

| Provider | IPv4 | IPv6 |
|----------|------|------|
| **Cloudflare** | `1.1.1.1`, `1.0.0.1` | `2606:4700:4700::1111`, `2606:4700:4700::1001` |
| **Google** | `8.8.8.8`, `8.8.4.4` | `2001:4860:4860::8888`, `2001:4860:4860::8888` |
| **Quad9** | `9.9.9.9`, `149.112.112.112` | `2620:fe::fe`, `2620:fe::9` |

Select at least one IPv4 and one IPv6 provider.

### 2. DNS Settings

| Setting | Recommended Value |
|---------|-------------------|
| Interface listening behavior | `Listen on all interfaces` |
| ☑ Use Conditional Forwarding | On (if you want local hostname resolution) |
| Local network | `172.20.0.0/24` (your Docker network) |

## Configure Local DNS Records (for `.internal` domains)

If you're using `.internal` domains (e.g., `sonarr.internal`), you need to add Local DNS records in Pi-hole to point them to your server's IP address on the local network.

### Step 1: Find Your Server's Local IP

Your server has an IP address on your local network (e.g., `192.168.1.100`). Find it with:

```bash
ip addr show | grep "inet " | grep -v "127.0.0.1"
```

Or check your router's DHCP client list. The IP will be something like `192.168.1.X`.

> **Important:** Use your server's actual local network IP (e.g., `192.168.1.100`), NOT the Docker container IP (172.20.0.X). Clients on your network need to reach the host machine, which then forwards traffic to NPM via Docker's port mapping.

### Step 2: Add Local DNS Records

Go to **Local DNS → DNS Records** in Pi-hole.

Add each domain pointing to your server's local IP:

| Domain | IP Address |
|--------|------------|
| `sonarr.internal` | `192.168.1.100` (your server's IP) |
| `radarr.internal` | `192.168.1.100` |
| `prowlarr.internal` | `192.168.1.100` |
| `bazarr.internal` | `192.168.1.100` |
| `qbittorrent.internal` | `192.168.1.100` |
| `jellyfin.internal` | `192.168.1.100` |
| `jellyseerr.internal` | `192.168.1.100` |
| `bindery.internal` | `192.168.1.100` |
| `calibre.internal` | `192.168.1.100` |
| `pdf.internal` | `192.168.1.100` |
| `pihole.internal` | `192.168.1.100` |

> **Replace `192.168.1.100` with your actual server IP.** All `.internal` domains point to the same IP because Nginx Proxy Manager (running on the host) routes traffic to the correct container based on the domain name.

### Step 3: Test DNS Resolution

From any device on your network:

```bash
ping sonarr.internal
```

It should resolve to your server's IP address (e.g., `192.168.1.100`).

### How Traffic Flows

```
Client Device (192.168.1.50)
    ↓
Requests: sonarr.internal
    ↓
Pi-hole DNS resolves to: 192.168.1.100
    ↓
Traffic goes to: Server (192.168.1.100)
    ↓
Docker port mapping: 80 → NPM container
    ↓
NPM routes to: sonarr:8989 (internal Docker network)
```

### Complete Container Reference Table

| Container | Domain | Server IP | NPM Port |
|-----------|--------|-----------|----------|
| Sonarr | `sonarr.internal` | `192.168.1.100` | `8989` |
| Radarr | `radarr.internal` | `192.168.1.100` | `7878` |
| Prowlarr | `prowlarr.internal` | `192.168.1.100` | `9696` |
| Bazarr | `bazarr.internal` | `192.168.1.100` | `6767` |
| qBittorrent | `qbittorrent.internal` | `192.168.1.100` | `8080` |
| Jellyfin | `jellyfin.internal` | `192.168.1.100` | `8096` |
| Jellyseerr | `jellyseerr.internal` | `192.168.1.100` | `5055` |
| Bindery | `bindery.internal` | `192.168.1.100` | `8787` |
| Calibre-Web | `calibre.internal` | `192.168.1.100` | `8083` |
| Stirling-PDF | `pdf.internal` | `192.168.1.100` | `8080` |
| Pi-hole | `pihole.internal` | `192.168.1.100` | `8443` |

> **Note:** All `.internal` domains point to your server's local IP. NPM then routes to the correct container based on the domain name.

## Configure Ad Lists

Go to **Adlists** to manage blocklists.

### Recommended Blocklists

| List | URL | Description |
|------|-----|-------------|
| **Steven Black** | `https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts` | General ads, malware, tracking |
| **OISD** | `https://oisd.nl/small` | Lightweight, low false positives |
| **1Hosts (Lite)** | `https://o0.pages.dev/Lite/adblock.txt` | Balanced protection |
| **HaGeZi** | `https://raw.githubusercontent.com/hagezi/dns-blocklists/main/wildcard-pro.txt` | Comprehensive protection |

### Add a Blocklist

1. Copy the URL of the blocklist
2. Go to **Adlists**
3. Paste the URL in the **Address** field
4. Click **Add**
5. Go to **Tools → Update Gravity** to apply

### Update Gravity

After adding blocklists:

1. Go to **Tools → Update Gravity**
2. Click **Update**
3. Wait for the process to complete

## Configure Whitelists

If legitimate sites are blocked, add them to the whitelist:

1. Go to **Whitelist**
2. Enter the domain (e.g., `example.com`)
3. Click **Add**

### Recommended Whitelists

| List | URL | Description |
|------|-----|-------------|
| **Common Whitelist** | `https://raw.githubusercontent.com/anudeepND/whitelist/master/domains/whitelist.txt` | Common false positives |

## Configure DHCP (Optional)

If you want Pi-hole to assign IP addresses:

Go to **Settings → DHCP**:

| Setting | Value |
|---------|-------|
| ☑ DHCP server enabled | On |
| Range of IP addresses | `192.168.1.100` - `192.168.1.200` |
| Router (gateway) | `192.168.1.1` |
| Lease time | `24` hours |

> **Note:** Disable your router's DHCP server if enabling this.

## Point Your Router to Pi-hole

To enable network-wide ad blocking:

1. Log into your router's admin panel
2. Go to **LAN Settings** or **DHCP Settings**
3. Set the **DNS Server** to your Pi-hole's IP address
4. Save and restart your router

All devices on your network will now use Pi-hole for DNS.

## Configure Per-Device DNS (Alternative)

If you don't want to change router settings:

1. On each device, manually set the DNS server to Pi-hole's IP
2. Or configure DHCP reservations with custom DNS

## View Query Logs

Go to **Query Log** to see all DNS queries:

- **Blocked queries**: Ads and trackers
- **Forwarded queries**: Legitimate traffic
- **Cached queries**: Previously resolved domains

## Configure Privacy

Go to **Settings → Privacy**:

| Setting | Recommended Value |
|---------|-------------------|
| Anonymous statistics | Off (or On if you want to contribute) |
| Query Log | Show everything (or your preference) |

## Next Steps

→ [Back to README](../README.md)

## Reference

- [Pi-hole Documentation](https://docs.pi-hole.net/)
- [Pi-hole Docker GitHub](https://github.com/pi-hole/docker-pi-hole)
- [Pi-hole Blocklist Collection](https://firebog.net/)
