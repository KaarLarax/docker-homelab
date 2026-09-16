# Calibre-Web Automated Setup Guide

Calibre-Web Automated (CWA) is an ebook library manager with a web interface. It automatically imports books from the ingest folder and manages the library with its own database.

In this stack, Calibre is the source of truth for the ebook library. Bindery downloads books and hands them off to Calibre via the ingest folder.

## Access the Web UI

### Via Direct IP (Initial Setup)

```
http://<your-server-ip>:8083
```

### Via NPM Domain (After NPM Setup)

```
http://calibre.internal
# or
https://calibre.yourdomain.com
```

## Initial Setup

### 1. First Login

Default credentials:
- **Username:** `admin`
- **Password:** `admin123`

Change the password immediately after first login.

### 2. Configure Library Location

On first login, you'll be asked for the Calibre library location:

| Setting | Value |
|---------|-------|
| Calibre Library Location | `/calibre-library` |

This maps to `./data/media/books/books` on your host.

> **Important:** This is where Calibre stores its database (`metadata.db`) and organized library. Do not point this to the ingest folder.

### 3. Basic Settings

Go to **Admin → Basic Configuration**:

| Setting | Value |
|---------|-------|
| Library Name | Your choice (e.g., "My Library") |
| ☑ Enable anonymous browsing | Off (recommended) |
| ☑ Enable registration | On/Off (your choice) |
| ☑ Allow uploads | On (if you want manual uploads) |
| ☑ Allow edits | On |

## Configure Automated Import

Calibre-Web Automated watches the ingest folder and automatically imports new books.

### Ingest Folder

The ingest folder is pre-configured in the compose file:

| Setting | Value |
|---------|-------|
| Ingest folder path | `/cwa-book-ingest` |

This maps to `./data/media/books/books-ingest` on your host.

### How It Works

1. Bindery downloads ebooks → `/data/torrents/books/books`
2. Bindery creates hardlink in `/data/media/books/books-ingest`
3. Calibre-Web Automated detects new files in ingest folder
4. Automatically imports them into the library
5. Organizes metadata, covers, and file structure
6. Updates `metadata.db`

### Manual Import

You can also trigger a manual import:

1. Go to **Admin → Tasks**
2. Click **Scan files for metadata**

## Configure Metadata Sources

Go to **Admin → Metadata Sources**:

### Recommended Sources

| Source | Priority | Notes |
|--------|----------|-------|
| **Google** | 1 | Best general coverage |
| **Amazon** | 2 | Good for commercial books |
| **Goodreads** | 3 | Community ratings |
| **Lublin** | 4 | Open library |

Enable the sources you want and set the order of preference.

## Write Integration

### Disable Write Integration

Go to **Settings → Write integration**:

| Setting | Value |
|---------|-------|
| Write integration | `Off` |

> **Why Off?** Calibre-Web Automated manages the library independently. There's no need to call external Calibre commands. CWA handles all imports and metadata updates internally.

### CWA Ingest Folder

The ingest folder is configured in the compose file:

| Setting | Value |
|---------|-------|
| Ingest folder path | `/data/media/books/books-ingest` |

This is where Bindery drops files for Calibre to import.

> **Mount the same path into both containers:** Both Bindery and Calibre-Web need access to this folder. The compose file already configures this:
> - Bindery: `../data:/data` (so it can write to `/data/media/books/books-ingest`)
> - Calibre-Web: `../data/media/books/books-ingest:/cwa-book-ingest`

## User Management

Go to **Admin → User Management**:

### Create Users

1. Click **New User**
2. Set username and password
3. Configure permissions:
   - ☑ Allow browsing
   - ☑ Allow downloading
   - ☑ Allow uploading (optional)
   - ☑ Allow editing (optional)
   - ☑ Admin rights (for admins only)

## Configure Kobo Integration (Optional)

If you have a Kobo e-reader:

1. Go to **Admin → Features → Kobo Sync**
2. ☑ **Enable Kobo Sync**
3. Configure sync settings
4. Connect your Kobo device using the sync URL

## Configure Email (Optional)

To send books via email:

Go to **Admin → Email Configuration**:

| Setting | Value |
|---------|-------|
| SMTP Server | Your SMTP server |
| SMTP Port | 587 (TLS) or 465 (SSL) |
| Username | Your email username |
| Password | Your email password |
| From Address | Your email address |

## How Calibre Manages the Library

Calibre maintains its own database (`metadata.db`) that tracks:
- Book metadata (title, author, publisher, etc.)
- File locations
- Covers
- Tags and categories
- Reading progress

When a new file arrives in the ingest folder:

1. CWA reads the file
2. Fetches metadata from configured sources
3. Creates a new entry in `metadata.db`
4. Organizes the file into Calibre's folder structure
5. Generates/updates cover images
6. Makes the book available in the web UI

> **Key point:** Never manually move or rename files in `/data/media/books/books/`. Always let Calibre manage the library structure.

## Next Steps

→ [Back to README](../README.md)

## Reference

- [Calibre-Web Automated GitHub](https://github.com/crocodilestick/Calibre-Web-Automated)
- [Calibre-Web Wiki](https://github.com/janeczku/calibre-web/wiki)
