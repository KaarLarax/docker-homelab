# Jellyseerr Setup Guide

Jellyseerr is a request management tool for Jellyfin. Users can browse, request, and track movies and TV shows.

## Access the Web UI

### Via Direct IP (Initial Setup)

```
http://<your-server-ip>:5055
```

### Via NPM Domain (After NPM Setup)

```
http://jellyseerr.internal
# or
https://jellyseerr.yourdomain.com
```

## Initial Setup

### 1. Sign In with Jellyfin

1. Click **Sign in with Jellyfin**
2. Enter your Jellyfin admin credentials:
   - Server URL: `http://jellyfin:8096`
   - Username: Your Jellyfin admin username
   - Password: Your Jellyfin admin password
3. Click **Sign In**

> **Important:** Use the same admin account you created in Jellyfin. This account will have full admin rights in Jellyseerr.

### 2. Configure Jellyfin Server

| Setting | Value |
|---------|-------|
| Jellyfin URL (Internal) | `http://jellyfin:8096` |
| Jellyfin URL (External) | `https://jellyfin.yourdomain.com` (your NPM domain) |

> **Why both URLs?** The internal URL is used for container-to-container communication. The external URL is given to users so they can access Jellyfin directly from their requests.

Click **Next**.

### 3. Scan Media Libraries

Jellyseerr will scan your Jellyfin libraries to know what's already available.

- Click **Scan Media Libraries**
- Wait for the scan to complete

## User Account Management

### How Users Sign In

Users have two ways to sign in to Jellyseerr:

#### Option A: Sign in with Jellyfin (Recommended)

1. User goes to Jellyseerr
2. Clicks **Sign in with Jellyfin**
3. Enters their Jellyfin credentials
4. Jellyseerr verifies with Jellyfin and logs them in

> **Pros:** Single sign-on, no extra passwords to remember, automatically syncs with Jellyfin user management.

#### Option B: Local Account

1. Admin creates a local user in Jellyseerr
2. User signs in with Jellyseerr-specific credentials

> **Pros:** Separate from Jellyfin, useful for users who don't have Jellyfin accounts yet.

### Import Users from Jellyfin

To import all Jellyfin users into Jellyseerr:

1. Go to **Settings → Users**
2. Click **Import from Jellyfin**
3. All Jellyfin users are now available in Jellyseerr

### Create Local Users (Alternative)

If you want users who don't have Jellyfin accounts:

1. Go to **Settings → Users → Create User**
2. Fill in:

| Setting | Value |
|---------|-------|
| Username | User's name |
| Email | User's email |
| Password | User's password |

3. Set permissions (see below)

### User Permissions

Edit a user to set their permissions:

| Permission | Description |
|------------|-------------|
| **Auto-approve movies** | User's movie requests are auto-approved (no admin approval needed) |
| **Auto-approve series** | User's TV requests are auto-approved |
| **Request limit** | Maximum number of pending requests (0 = unlimited) |
| **Enable 4K requests** | Allow requesting 4K content |
| **Manage requests** | Can approve/reject other users' requests |
| **Admin** | Full access to all settings |

### User Roles

| Role | Capabilities |
|------|-------------|
| **Admin** | Full access, can approve requests, manage users, change settings |
| **Manage Requests** | Can approve/reject requests from other users |
| **User** | Can request content, view own requests |

## Connect to Radarr

Go to **Settings → Services → Add Radarr Server**:

| Setting | Value |
|---------|-------|
| Name | `Radarr` |
| Default Server | ☑ On |
| Hostname | `radarr` |
| Port | `7878` |
| API Key | *(from Radarr → Settings → General → API Key)* |
| SSL | Off |
| Base URL | Leave empty |
| Root Folder | `/data/media/movies` |
| Quality Profile | Select your profile |
| Minimum Availability | `Released` (or your preference) |
| Tags | Optional |

Click **Test** → **Save**.

## Connect to Sonarr

Go to **Settings → Services → Add Sonarr Server**:

| Setting | Value |
|---------|-------|
| Name | `Sonarr` |
| Default Server | ☑ On |
| Hostname | `sonarr` |
| Port | `8989` |
| API Key | *(from Sonarr → Settings → General → API Key)* |
| SSL | Off |
| Base URL | Leave empty |
| Root Folder | `/data/media/tv` |
| Quality Profile | Select your profile |
| Language Profile | `English` (or your preference) |
| Series Type | `Standard` |
| Season Folders | ☑ On |
| Tags | Optional |
| Anime Root Folder | (optional, if you have anime) |
| Anime Quality Profile | (optional) |
| Anime Tags | (optional) |

Click **Test** → **Save**.

## Configure Notifications (Optional)

Go to **Settings → Notifications**:

### Supported Notification Services

| Service | Use Case |
|---------|----------|
| **Discord** | Send request notifications to a Discord channel |
| **Telegram** | Send notifications to Telegram |
| **Email (SMTP)** | Email notifications |
| **Pushbullet** | Push notifications |
| **Webhook** | Custom webhook integrations |

### Example: Discord Setup

1. Go to **Settings → Notifications → Discord**
2. Enter your Discord webhook URL
3. Select events to notify:
   - ☑ New request
   - ☑ Request approved
   - ☑ Request available
4. Click **Save**

## Configure Jobs

Go to **Settings → Jobs**:

| Job | Recommended Schedule |
|-----|---------------------|
| Full Media Scan | Every 24 hours |
| Process Radarr/Sonarr | Every 5 minutes |
| Rebuild Jellyfin | Every 24 hours |
| Availability Sync | Every 6 hours |

## Next Steps

→ [Set up Bindery](11-bindery.md)

## Reference

- [Jellyseerr Documentation](https://docs.jellyseerr.dev/)
- [Jellyseerr GitHub](https://github.com/Fallenbagel/jellyseerr)
