# Sonarr Setup Guide

Sonarr manages your TV show collection. It monitors, grabs, and organizes TV series automatically.

## Access the Web UI

### Via Direct IP (Initial Setup)

```
http://<your-server-ip>:8989
```

### Via NPM Domain (After NPM Setup)

```
http://sonarr.internal
# or
https://sonarr.yourdomain.com
```

## Initial Setup Wizard

1. **Authentication** → Set credentials or skip
2. **Folder Management** → Skip (we'll configure manually)

## Configure Media Management

### 1. Root Folder

Go to **Settings → Media Management → Root Folders → Add Root Folder**:

| Setting | Value |
|---------|-------|
| Name | `TV` |
| Path | `/data/media/tv` |

### 2. Naming Convention

Go to **Settings → Media Management → Episode Naming**:

| Setting | Recommended Value |
|---------|-------------------|
| ☑ Rename Episodes | On |
| ☑ Replace Illegal Characters | On |
| Standard Episode Format | `{Series Title} - S{season:00}E{episode:00} - {Episode Title} [{Quality Title}]` |
| Daily Episode Format | `{Series Title} - {Air-Date} - {Episode Title} [{Quality Title}]` |
| Series Folder Format | `{Series Title}` |
| Season Folder Format | `Season {season:00}` |

> Follow [Trash Guides naming conventions](https://trash-guides.info/Sonarr/Sonarr-recommended-naming-scheme/) for best results.

### 3. Import Settings

| Setting | Value |
|---------|-------|
| ☑ Use Hardlinks instead of Copy | On |
| ☑ Import Extra Files | On (srt, nfo, jpg, png) |

## Connect Download Client

Go to **Settings → Download Clients → Add Download Client**:

| Setting | Value |
|---------|-------|
| Name | `qBittorrent` |
| Host | `qbittorrent` |
| Port | `8080` |
| Username | Your qBittorrent username |
| Password | Your qBittorrent password |
| Category | `tv` |

Click **Test** → **Save**.

## Configure Language

### Set Original Language Preference

Go to **Settings → Profiles → Languages**:

| Setting | Value |
|---------|-------|
| Language | `Original` |

This ensures Sonarr grabs releases in the original language of the show.

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

> **How it works:** Higher scores give priority to Spanish releases. A score of `+10` means Sonarr will prefer Spanish releases over non-Spanish ones of the same quality.

### Import Trash Guides Custom Formats

For the best quality selection, import custom formats from Trash Guides:

1. Go to **Settings → Custom Formats**
2. Import from [Trash Guides: Sonarr Custom Formats](https://trash-guides.info/Sonarr/Sonarr-import-custom-formats/)
3. Apply them to your quality profiles

## Configure Quality Profiles

Go to **Settings → Profiles → Quality Profiles**.

### Recommended: Create a Custom Profile

Follow [Trash Guides: Sonarr Quality Settings](https://trash-guides.info/Sonarr/Sonarr-Quality-Settings-File-Size/) for detailed quality profiles.

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

2. **Set Minimum File Size**:
   - Minimum: `50 MB` per episode for 720p, `100 MB` for 1080p

## Add TV Shows

1. Click **Series → Add New**
2. Search for a show by name
3. Select:
   - **Quality Profile**: Your custom profile
   - **Root Folder**: `/data/media/tv`
   - **Season Folders**: On
4. Click **Add**

Sonarr will automatically search for and download episodes.

## Connect to Bazarr (Subtitles)

After setting up Bazarr, connect it:

1. In Bazarr → **Settings → Sonarr**
2. Enter:
   - Host: `sonarr`
   - Port: `8989`
   - API Key: from Sonarr → **Settings → General → API Key**

## Connect to Jellyseerr (Requests)

1. In Jellyseerr → **Settings → Services → Sonarr**
2. Enter:
   - Hostname: `sonarr`
   - Port: `8989`
   - API Key: from Sonarr → **Settings → General → API Key**
   - Root Folder: `/data/media/tv`
   - Quality Profile: Select your profile

## Next Steps

→ [Set up Bazarr](08-bazarr.md)

## Reference

- [Trash Guides: Sonarr](https://trash-guides.info/Sonarr/)
- [Trash Guides: Quality Settings](https://trash-guides.info/Sonarr/Sonarr-Quality-Settings-File-Size/)
- [Trash Guides: Custom Formats](https://trash-guides.info/Sonarr/Sonarr-import-custom-formats/)
