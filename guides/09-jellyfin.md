# Jellyfin Setup Guide

Jellyfin is your media server. It streams your movies, TV shows, and music to all your devices.

## Access the Web UI

### Via Direct IP (Initial Setup)

```
http://<your-server-ip>:8096
```

### Via NPM Domain (After NPM Setup)

```
http://jellyfin.internal
# or
https://jellyfin.yourdomain.com
```

## Initial Setup Wizard

### 1. Language & Country

Select your preferred language and country.

### 2. Create Admin User

| Setting | Value |
|---------|-------|
| Username | Your choice |
| Password | Your choice (strong password recommended) |

> **Important:** This admin user will be used by Jellyseerr to authenticate. Remember these credentials.

### 3. Add Media Libraries

#### Movies Library

| Setting | Value |
|---------|-------|
| Name | `Movies` |
| Content Type | `Movies` |
| Folders | `/Media/movies` |
| Language | Your preference |
| Country | Your preference |

#### TV Shows Library

| Setting | Value |
|---------|-------|
| Name | `TV Shows` |
| Content Type | `Shows` |
| Folders | `/Media/tv` |
| Language | Your preference |
| Country | Your preference |

### 4. Metadata Language

| Setting | Value |
|---------|-------|
| Preferred language | `English` (or your preference) |
| Country | Your country |

### 5. Remote Access

Skip this if using Nginx Proxy Manager (recommended).

## Configure Hardware Transcoding

Your Jellyfin container is configured with `/dev/dri` for hardware acceleration.

### Enable Hardware Transcoding

1. Go to **Dashboard → Playback → Transcoding**
2. Set:

| Setting | Value |
|---------|-------|
| Hardware acceleration | `Video Acceleration API (VA-API)` |
| VA-API Device | `/dev/dri/renderD128` |
| ☑ Enable hardware decoding for | H264, HEVC, VP9, AV1 (select what your GPU supports) |
| ☑ Enable hardware encoding | On (if supported) |
| ☑ Enable tonemapping | On (for HDR → SDR) |

### Check GPU Support

To see what your GPU supports:

```bash
docker exec jellyfin vainfo
```

Look for supported codecs under `VAProfile*`.

## Configure DLNA (Optional)

If you want to use DLNA for local device discovery:

1. Go to **Dashboard → DLNA**
2. ☑ **Enable DLNA**
3. Ports `1900` and `7359` are already exposed in the compose file

## Set Up Users

Go to **Dashboard → Users**.

### Create User Accounts

1. Click **Add User**
2. Fill in:

| Setting | Value |
|---------|-------|
| Username | User's name |
| Password | User's password |
| ☑ Allow remote access | On (if using NPM) |

3. Configure permissions:

| Permission | Recommended |
|------------|-------------|
| ☑ Allow media playback | On |
| ☑ Allow remote access | On |
| ☑ Allow content deletion | Off |
| ☑ Allow managing libraries | Off |
| ☑ Allow managing users | Off |

### User Access to Libraries

By default, new users have access to all libraries. To restrict:

1. Edit the user
2. Go to **Access** tab
3. Uncheck libraries the user shouldn't access

### Parental Controls

For child accounts:

1. Edit the user
2. Go to **Parental Control** tab
3. Set restrictions:
   - Maximum allowed rating
   - Block items with unrated or unrecognized items
   - Set allowed tags

## Install Plugins (Optional)

Go to **Dashboard → Plugins → Catalog**:

### Recommended Plugins

| Plugin | Description |
|--------|-------------|
| **Open Subtitles** | Direct subtitle search within Jellyfin |
| **Fanart** | Additional artwork |
| **Intro Skipper** | Skip intros/recaps automatically |
| **Trakt** | Sync watch history with Trakt.tv |

## Connect to Jellyseerr

Jellyseerr uses Jellyfin for authentication. Users sign in to Jellyseerr with their Jellyfin accounts.

### Configure Jellyseerr Connection

1. In Jellyseerr → **Settings → Media Servers → Jellyfin**
2. Enter:

| Setting | Value |
|---------|-------|
| Jellyfin URL (Internal) | `http://jellyfin:8096` |
| Jellyfin URL (External) | `https://jellyfin.yourdomain.com` (your NPM domain) |
| Username | Your Jellyfin admin username |
| Password | Your Jellyfin admin password |

3. Click **Test** → **Save**

### How User Authentication Works

1. User goes to Jellyseerr (`jellyseerr.internal` or `jellyseerr.yourdomain.com`)
2. Clicks **Sign in with Jellyfin**
3. Enters their Jellyfin credentials
4. Jellyseerr verifies with Jellyfin
5. User is logged in and can request content

### Sync Users Between Jellyfin and Jellyseerr

Jellyseerr automatically syncs users from Jellyfin:

1. Go to **Jellyseerr → Settings → Users**
2. Click **Import from Jellyfin**
3. All Jellyfin users are now available in Jellyseerr

### Set User Permissions in Jellyseerr

After importing users:

1. Edit a user in Jellyseerr
2. Set permissions:

| Permission | Description |
|------------|-------------|
| **Auto-approve movies** | User's movie requests are auto-approved |
| **Auto-approve series** | User's TV requests are auto-approved |
| **Request limit** | Maximum number of pending requests |
| **Enable 4K requests** | Allow requesting 4K content |

## Next Steps

→ [Set up Jellyseerr](10-jellyseerr.md)

## Reference

- [Jellyfin Documentation](https://jellyfin.org/docs/)
- [Jellyfin Hardware Transcoding](https://jellyfin.org/docs/general/administration/hardware-acceleration/)
