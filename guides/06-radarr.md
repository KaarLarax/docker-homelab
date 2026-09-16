# Radarr Setup Guide

Radarr manages your movie collection. It monitors, grabs, and organizes movies automatically.

## Access the Web UI

### Via Direct IP (Initial Setup)

```
http://<your-server-ip>:7878
```

### Via NPM Domain (After NPM Setup)

```
http://radarr.internal
# or
https://radarr.yourdomain.com
```

## Initial Setup Wizard

1. **Authentication** → Set credentials or skip
2. **Folder Management** → Skip (we'll configure manually)

## Configure Media Management

### 1. Root Folder

Go to **Settings → Media Management → Root Folders → Add Root Folder**:

| Setting | Value |
|---------|-------|
| Name | `Movies` |
| Path | `/data/media/movies` |

### 2. Naming Convention

Go to **Settings → Media Management → Movie Naming**:

| Setting | Recommended Value |
|---------|-------------------|
| ☑ Rename Movies | On |
| ☑ Replace Illegal Characters | On |
| Standard Movie Format | `{Movie Title} ({Release Year}) {Quality Title} {MediaFull}` |
| Movie Folder Format | `{Movie Title} ({Release Year})` |

> Follow [Trash Guides naming conventions](https://trash-guides.info/Radarr/Radarr-recommended-naming-scheme/) for best results.

### 3. Import Settings

| Setting | Value |
|---------|-------|
| ☑ Use Hardlinks instead of Copy | On |
| ☑ Import Extra Files | On (srt, nfo, jpg, png) |

> **Important:** Hardlinks only work when torrents and media are on the same filesystem. Since both are under `/data/`, this should work.

## Connect Download Client

Go to **Settings → Download Clients → Add Download Client**:

| Setting | Value |
|---------|-------|
| Name | `qBittorrent` |
| Host | `qbittorrent` |
| Port | `8080` |
| Username | Your qBittorrent username |
| Password | Your qBittorrent password |
| Category | `movies` |

Click **Test** → **Save**.

## Configure Language

### Set Original Language Preference

Go to **Settings → Profiles → Languages**:

| Setting | Value |
|---------|-------|
| Language | `Original` |

This ensures Radarr grabs releases in the original language of the movie.

### Spanish Language Custom Formats

To prioritize Spanish audio or include Spanish releases, create custom formats.

Go to **Settings → Custom Formats → Add Custom Format**:

#### Custom Format: Spanish Audio

| Setting | Value |
|---------|-------|
| Name | `Spanish` |
| Specifications → Add Condition | |
| Type | `Language` |
| Language | `Spanish` |
| Condition | `Language` |

#### Custom Format: Spanish Latino

| Setting | Value |
|---------|-------|
| Name | `Spanish Latino` |
| Specifications → Add Condition | |
| Type | `Language` |
| Language | `Spanish` |
| Condition | `Language` |

#### Custom Format: Spanish Castilian

| Setting | Value |
|---------|-------|
| Name | `Spanish Castilian` |
| Specifications → Add Condition | |
| Type | `Language` |
| Language | `Spanish` |
| Condition | `Language` |

> **Tip:** You can also use regex-based custom formats to match specific release group names or tags that indicate Spanish audio.

### Apply Custom Formats to Quality Profiles

Go to **Settings → Profiles → Quality Profiles** and edit your profile:

1. Scroll to **Custom Formats** section
2. Add your Spanish custom formats
3. Set scores:

| Custom Format | Score |
|---------------|-------|
| `Spanish` | `+10` |
| `Spanish Latino` | `+5` |
| `Spanish Castilian` | `+5` |

> **How it works:** Higher scores give priority to Spanish releases. A score of `+10` means Radarr will prefer Spanish releases over non-Spanish ones of the same quality.

### Import Trash Guides Custom Formats

For the best quality selection, import custom formats from Trash Guides:

1. Go to **Settings → Custom Formats**
2. Import the recommended CFs from [Trash Guides: Radarr Custom Formats](https://trash-guides.info/Radarr/Radarr-import-custom-formats/)
3. Apply them to your quality profiles

## Configure Quality Profiles

Go to **Settings → Profiles → Quality Profiles**.

### Recommended: Create a Custom Profile

Follow [Trash Guides: Radarr Quality Settings](https://trash-guides.info/Radarr/Radarr-Quality-Settings-File-Size/) for detailed quality profiles.

Quick setup:

1. **Edit "Any"** profile (or create a new one):
   - Set **Quality Groups**:
     - `Bluray-2160p` (if 4K)
     - `Bluray-1080p`
     - `WEB 1080p`
     - `WEB 720p`
     - `HDTV 1080p`
   - Set **Language**: Original or Spanish (if you prefer Spanish releases)
   - Set **Upgrade Until**: `Bluray-2160p` or `Bluray-1080p`

2. **Set Minimum File Size** (to avoid CAM/TS):
   - Minimum: `50 MB` for 720p, `100 MB` for 1080p

## Add Movies

1. Click **Movies → Add New**
2. Search for a movie by name
3. Select quality profile
4. Set root folder: `/data/media/movies`
5. Click **Add**

Radarr will automatically search for and download the movie.

## Connect to Bazarr (Subtitles)

After setting up Bazarr, connect it:

1. In Bazarr → **Settings → Radarr**
2. Enter:
   - Host: `radarr`
   - Port: `7878`
   - API Key: from Radarr → **Settings → General → API Key**

## Connect to Jellyseerr (Requests)

1. In Jellyseerr → **Settings → Services → Radarr**
2. Enter:
   - Hostname: `radarr`
   - Port: `7878`
   - API Key: from Radarr → **Settings → General → API Key**
   - Root Folder: `/data/media/movies`
   - Quality Profile: Select your profile

## Next Steps

→ [Set up Sonarr](07-sonarr.md)

## Reference

- [Trash Guides: Radarr](https://trash-guides.info/Radarr/)
- [Trash Guides: Quality Settings](https://trash-guides.info/Radarr/Radarr-Quality-Settings-File-Size/)
- [Trash Guides: Custom Formats](https://trash-guides.info/Radarr/Radarr-import-custom-formats/)
