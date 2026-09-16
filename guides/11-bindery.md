# Bindery Setup Guide

Bindery automates audiobook and ebook downloads. It monitors your library and downloads new releases automatically.

In this stack, Bindery works in **External mode** -- it downloads books and hands them off to Calibre-Web Automated, which manages the library and metadata.

## Access the Web UI

### Via Direct IP (Initial Setup)

```
http://<your-server-ip>:8787
```

### Via NPM Domain (After NPM Setup)

```
http://bindery.internal
# or
https://bindery.yourdomain.com
```

## Initial Configuration

### 1. Set Up Authentication

On first launch, create an admin account:

| Setting | Value |
|---------|-------|
| Username | Your choice |
| Password | Your choice |

### 2. Configure Library Paths

Bindery is pre-configured with these paths in the compose file:

| Setting | Value |
|---------|-------|
| Library Directory | `/data/media/books/books` |
| Audiobook Directory | `/data/media/books/audiobooks` |
| Download Directory | `/data/torrents/books/books` |
| Audiobook Download Directory | `/data/torrents/books/audiobooks` |

Verify these paths are correct in the Bindery settings.

### 3. Connect to qBittorrent

Go to **Settings → Downloaders → qBittorrent**:

| Setting | Value |
|---------|-------|
| Host | `qbittorrent` |
| Port | `8080` |
| Username | Your qBittorrent username |
| Password | Your qBittorrent password |
| Category (Books) | `books` |
| Category (Audiobooks) | `audiobooks` |

Click **Test** → **Save**.

### 4. Configure Indexers

Go to **Settings → Indexers**:

Bindery can use Prowlarr or direct indexer connections.

#### Option A: Use Prowlarr (Recommended)

| Setting | Value |
|---------|-------|
| Indexer Type | `Prowlarr` |
| Prowlarr URL | `http://prowlarr:9696` |
| API Key | *(from Prowlarr → Settings → General → API Key)* |

#### Option B: Direct Indexer

| Setting | Value |
|---------|-------|
| Indexer Type | Direct |
| Indexer URL | Your indexer's URL |
| API Key | Your indexer API key |

### 5. Configure File Naming & Import Mode

This is the most important part -- Bindery needs to hand off downloads to Calibre-Web Automated.

Go to **Settings → File Naming**:

#### External File Naming

| Setting | Value |
|---------|-------|
| File Naming | `External` |

> **Why External?** Calibre-Web Automated manages the library with its own database. Bindery should not rename or organize files -- it should just download them and let Calibre handle the rest.

#### Import Mode

| Setting | Value |
|---------|-------|
| Import Mode | `External` |

**Import Mode options explained:**

| Mode | Description | When to Use |
|------|-------------|-------------|
| **Auto** | Hardlinks when possible, copies otherwise | When Bindery manages the library directly |
| **Move** | Moves the source file (breaks seeding!) | Never for torrents |
| **Copy** | Copies the file, keeps source | When Bindery manages the library |
| **Hardlink** | Creates hardlink, requires same filesystem | When Bindery manages the library |
| **External** | Hands off to another tool (Calibre, Grimmory) | **This stack** -- Calibre manages the library |

> **Why External?** In External mode, Bindery grabs the download and stops. Your tool (Calibre) processes it, then Bindery reconciles on the next library scan. This prevents conflicts between Bindery and Calibre's database.

#### Drop Folder

| Setting | Value |
|---------|-------|
| Drop folder | `/data/media/books/books-ingest` |

> **What is this?** In External mode, Bindery renames each finished download into this folder for Calibre-Web Automated to ingest. The source is never moved, so torrents keep seeding. Bindery still reconciles the managed copy on the next library scan.

#### Layout & Placement

| Setting | Value |
|---------|-------|
| Layout | `Flat` (file in folder root) |
| Placement | `Hardlink` (same filesystem) |

> **Layout Flat:** Files are placed directly in the ingest folder without subdirectories. Calibre will organize them into its own structure.

> **Placement Hardlink:** Uses hardlinks when the download folder and library share a volume. Since both are under `/data/`, this works and saves disk space.

### 6. Configure Monitoring

Go to **Settings → Monitoring**:

| Setting | Value |
|---------|-------|
| ☑ Auto-download new releases | On |
| Check interval | `24` hours |
| Preferred quality | Your preference |

### 7. Add Books to Monitor

Go to **Library → Add Book**:

1. Search for a book or audiobook
2. Select format (ebook or audiobook)
3. Choose quality preference
4. Click **Add**

Bindery will automatically download and organize the book into your library.

## How the Bindery → Calibre Pipeline Works

```
1. Bindery searches indexers for a book
2. Download starts in qBittorrent → /data/torrents/books/
3. Download completes
4. Bindery creates hardlink in /data/media/books/books-ingest/
5. Calibre-Web Automated detects new file in ingest folder
6. Calibre imports the book, adds metadata, organizes library
7. Book appears in Calibre-Web library at /data/media/books/books/
8. Bindery reconciles on next scan
```

> **Key point:** The torrent keeps seeding from `/data/torrents/books/` while Calibre manages the organized copy in `/data/media/books/books/`.

## File Organization

Calibre-Web Automated organizes files as follows:

```
data/media/books/
├── books/              # Ebooks (managed by Calibre)
│   ├── Author Name/
│   │   ├── Book Title/
│   │   │   ├── metadata.opf
│   │   │   ├── cover.jpg
│   │   │   └── Book Title.epub
├── books-ingest/       # Drop folder for Bindery → Calibre
│   └── (temporary files before Calibre imports them)
└── audiobooks/         # Audiobooks (managed by Bindery directly)
    ├── Author Name/
    │   ├── Book Title/
    │   │   ├── Chapter 01.mp3
    │   │   ├── Chapter 02.mp3
    │   │   └── cover.jpg
```

## Next Steps

→ [Set up Calibre-Web](12-calibre-web.md)

## Reference

- [Bindery GitHub](https://github.com/vavallee/bindery)
