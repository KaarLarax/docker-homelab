# Bazarr Setup Guide

Bazarr manages subtitles for your movies and TV shows. It automatically downloads and syncs subtitles based on your Radarr and Sonarr libraries.

## Access the Web UI

### Via Direct IP (Initial Setup)

```
http://<your-server-ip>:6767
```

### Via NPM Domain (After NPM Setup)

```
http://bazarr.internal
# or
https://bazarr.yourdomain.com
```

## Initial Setup

### 1. Set Authentication

Go to **Settings → General → Authentication**:

| Setting | Value |
|---------|-------|
| Authentication Type | `Form` |
| Username | Your choice |
| Password | Your choice |

### 2. Configure Base URL (if using reverse proxy)

If accessing through Nginx Proxy Manager:

| Setting | Value |
|---------|-------|
| Base URL | Leave empty (handled by NPM) |

## Connect to Sonarr

Go to **Settings → Sonarr**:

| Setting | Value |
|---------|-------|
| Host | `sonarr` |
| Port | `8989` |
| API Key | *(from Sonarr → Settings → General → API Key)* |
| Full Path | `/data/media/tv` |
| SSL | Off |

Click **Test** → **Save**.

## Connect to Radarr

Go to **Settings → Radarr**:

| Setting | Value |
|---------|-------|
| Host | `radarr` |
| Port | `7878` |
| API Key | *(from Radarr → Settings → General → API Key)* |
| Full Path | `/data/media/movies` |
| SSL | Off |

Click **Test** → **Save**.

## Configure Subtitle Providers

Go to **Settings → Providers → Add Provider**.

### Recommended Providers

| Provider | Type | Notes |
|----------|------|-------|
| **OpenSubtitles.com** | Free/Premium | Most popular, requires account. Premium gets higher download limits. |
| **SubDivX** | Free | Spanish subtitles (Latin American & Castilian) |
| **Addic7ed** | Free | Good for TV shows |
| **Podnapisi** | Free | Multi-language |
| **TVSubtitles** | Free | TV shows |
| **Subscene** | Free | Large collection, community-driven |

### Add OpenSubtitles.com

1. Click **+ Add Provider** → **OpenSubtitles.com**
2. Enter your OpenSubtitles.com username and password
3. Click **Save**

### Add SubDivX (Spanish Subtitles)

1. Click **+ Add Provider** → **SubDivX**
2. No account required
3. Click **Save**

### Add More Providers

Repeat for each provider you want to use. More providers = more subtitle availability.

## Configure Languages

Go to **Settings → Languages**.

### Subtitle Languages

Add the languages you want subtitles for:

| Language | Code | Notes |
|----------|------|-------|
| **English** | `en` | Default |
| **Spanish** | `es` | Spanish subtitles |
| **Spanish (Latino)** | `es-MX` | Latin American Spanish |
| **Spanish (Castilian)** | `es-ES` | Spain Spanish |

Click **Add** for each language.

### Language Profiles

Create a profile for each language you want:

#### Profile: English

1. Go to **Settings → Languages → Language Profiles**
2. Click **+ Add Language Profile**
3. Name: `English`
4. Add your subtitle providers in order of preference:
   - OpenSubtitles.com
   - Addic7ed
   - Podnapisi
5. Set cutoff: `OpenSubtitles.com`

#### Profile: Spanish

1. Click **+ Add Language Profile**
2. Name: `Spanish`
3. Add providers with good Spanish support:
   - SubDivX (best for Spanish)
   - OpenSubtitles.com
   - Podnapisi
4. Set cutoff: `SubDivX`

#### Profile: Spanish Latino

1. Click **+ Add Language Profile**
2. Name: `Spanish Latino`
3. Add providers:
   - SubDivX
   - OpenSubtitles.com
4. Set cutoff: `SubDivX`

### Apply Language Profiles to Series/Movies

When adding series or movies in Sonarr/Radarr, you can specify which language profile to use. Bazarr will then download subtitles for that language.

## Configure Subtitle Options

Go to **Settings → Subtitles**:

| Setting | Recommended Value |
|---------|-------------------|
| Subtitle Folder | `Subs` |
| Subtitle Naming | `{Series Title} - S{season:00}E{episode:00} - {Episode Title}` |
| ☑ Embedded Subtitles | On (use embedded subs if available) |
| ☑ External Subtitles | On |
| Subtitle Format | `srt` |
| ☑ Remove Old Subtitles | On (when upgrading) |

### Spanish Subtitle Naming

For Spanish subtitles, you may want to include the language code in the filename:

| Setting | Value |
|---------|-------|
| Multi-language subtitle naming | `{title}.{language}` |

This creates files like:
- `Movie Title (2024).en.srt` (English)
- `Movie Title (2024).es.srt` (Spanish)
- `Movie Title (2024).es-MX.srt` (Spanish Latino)

## Configure Anti-Captcha (Optional)

Some providers require solving CAPTCHAs. If needed:

Go to **Settings → Anti-Captcha**:

| Setting | Value |
|---------|-------|
| Anti-Captcha Provider | `Anti-Captcha` or `2Captcha` |
| API Key | Your API key from the provider |

## Test Subtitle Search

1. Go to **Movies** or **Series** in Bazarr
2. Click on a movie/show
3. Click **Search** for subtitles
4. Select the language profile (English, Spanish, etc.)
5. Check the logs to verify subtitles are being found and downloaded

## Next Steps

→ [Set up Jellyfin](09-jellyfin.md)

## Reference

- [Trash Guides: Bazarr](https://trash-guides.info/Bazarr/)
- [Bazarr Wiki](https://wiki.bazarr.media/)
