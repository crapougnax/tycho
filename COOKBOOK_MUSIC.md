# Sovereign Music Cookbook: The Nextcloud-Jellyfin Bridge

This cookbook explains how to create a seamless, high-performance music streaming setup by bridging **Nextcloud** (for synchronization) and **Jellyfin** (for streaming) using Sovereign's storage principles.

## The Principle

Instead of having isolated "silos" for each app, we use a **Shared Data Bridge**. 
- **Nextcloud** acts as the *Synchronizer*: You upload your music from your phone or PC via Nextcloud/WebDAV.
- **Jellyfin** acts as the *Player*: It points directly to the Nextcloud storage folder on the host to index and play the music without any duplication.

## Architecture

1.  **Physical Location**: All data resides in `${BASE_STORAGE_PATH}/nextcloud/data/data/${NC_USER}/files/Music`.
2.  **The Bridge**: We mount the Nextcloud data directory into the Jellyfin container as a read-only volume.

## Step-by-Step Implementation

### 1. Identify the Sync Folder
Ensure your music is synchronized in a dedicated folder in Nextcloud (e.g., `Music/`).

### 2. Configure the Jellyfin Recipe
In your Jellyfin `compose.yaml`, add a bind mount to the Nextcloud data directory:

```yaml
services:
  jellyfin:
    # ...
    volumes:
      - ${BASE_STORAGE_PATH}/jellyfin/config:/config
      - ${BASE_STORAGE_PATH}/jellyfin/cache:/cache
      # THE BRIDGE:
      - ${BASE_STORAGE_PATH}/nextcloud/data/data/crapougnax/files/Music:/media/Music:ro
```
*Note: Using `:ro` (read-only) ensures Jellyfin cannot accidentally modify or delete your sync files.*

### 3. Permissions Management
In a rootless Podman environment, ensure that the Jellyfin container has read access to the Nextcloud folders. 
Sovereign handles this by ensuring both apps run under the same user space, making UID/GID mapping consistent across containers.

### 4. Setup Jellyfin Library
1. Open your Jellyfin Dashboard.
2. Add a new Library of type **Music**.
3. Set the path to `/media/Music`.
4. Let Jellyfin scan the metadata.

## Benefits of this Setup

- **Zero Duplication**: One file, two uses. Save disk space.
- **Bi-directional Sync**: Any song added via Nextcloud appears instantly in Jellyfin after a scan.
- **Sovereign Security**: Files remain in your private storage space (`BASE_STORAGE_PATH`), isolated from other system users.

---
*Created as part of the Sovereign Project Principles.*
