# FlareSolverr Setup Guide

FlareSolverr is a proxy server that bypasses Cloudflare and DDoS-Guard protection. It's used by Prowlarr to access Cloudflare-protected indexers.

## How It Works

FlareSolverr runs as a background service. Prowlarr sends requests to FlareSolverr, which solves the Cloudflare challenge and returns the response.

**No web UI** -- FlareSolverr is a headless service.

## Verify It's Running

Check the container logs:

```bash
docker compose logs flaresolverr
```

You should see:
```
FlareSolverr is ready!
```

## Connect to Prowlarr

### 1. Get FlareSolverr URL

The internal Docker URL is:
```
http://flaresolverr:8191
```

### 2. Add to Prowlarr

1. Go to Prowlarr → **Settings → Apps → Add FlareSolverr**
2. Fill in:

| Setting | Value |
|---------|-------|
| Name | `FlareSolverr` |
| Host | `flaresolverr` |
| Port | `8191` |
| ☑ Use FlareSolverr | On |

3. Click **Test** → **Save**

### 3. Assign FlareSolverr to Indexers

For each Cloudflare-protected indexer:

1. Go to **Indexers → Edit Indexer**
2. Under **FlareSolverr**, select your FlareSolverr instance
3. Click **Save**

## Test FlareSolverr

To verify FlareSolverr is working:

1. In Prowlarr, go to **Indexers**
2. Find a Cloudflare-protected indexer
3. Click **Test**
4. Check the logs -- it should show FlareSolverr solving the challenge

## Troubleshooting

### FlareSolverr Not Responding

Check the logs:
```bash
docker compose logs -f flaresolverr
```

Common issues:
- Container not started: `docker compose up -d flaresolverr`
- Port conflict: Ensure port 8191 is not in use

### Indexer Still Blocked

Some indexers use advanced Cloudflare protection that FlareSolverr can't bypass. Try:
1. Restart FlareSolverr: `docker compose restart flaresolverr`
2. Update FlareSolverr: `docker compose pull flaresolverr && docker compose up -d flaresolverr`
3. Check if the indexer is temporarily down

### High Memory Usage

FlareSolverr uses a headless browser, which can be memory-intensive. If you're low on RAM:
- Limit concurrent requests in Prowlarr
- Restart FlareSolverr periodically

## Next Steps

→ [Set up Prowlarr](05-prowlarr.md) to manage your indexers

## Reference

- [FlareSolverr GitHub](https://github.com/FlareSolverr/FlareSolverr)
- [FlareSolverr Wiki](https://github.com/FlareSolverr/FlareSolverr/wiki)
