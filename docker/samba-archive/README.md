# Archive and File Access Container

This container provides SMB access to archived Time Machine backups and general
file storage such as photos.

It is intentionally separate from the Time Machine container to avoid mixing
sync-heavy backup workloads with normal file access.

## Purpose

- Provide read-only access to archived Time Machine backups
- Provide writable SMB access for photos and general files
- Avoid exposing active Time Machine sparsebundles

## Implementation Details

The container uses the `dperson/samba` image and relies on a custom `smb.conf`
mounted from the host.

This approach was chosen after encountering limitations with passing Samba
options via container entrypoint arguments.

Mounting a full configuration file provides predictable and stable behavior.

## SMB Configuration

The container mounts a custom configuration file at:

/etc/samba/smb.conf

This configuration defines two shares:

1. TM-Archive
   - Read-only
   - Exposes archived sparsebundles
   - Prevents accidental modification of backup history

2. Photos
   - Writable
   - Used for general file storage and browsing
   - Accessible from macOS Finder

Both shares explicitly force user and group mapping to avoid permission
mismatches between macOS and the Linux host.

## Storage Mapping

The entire archival SSD is mounted into the container:

/mnt/ssd → /data

Subdirectories are used to separate backups from general data.

## Permission Handling

The container runs as root, as required by Samba initialization.

File ownership and permissions are enforced at the filesystem level on the host,
with Samba configured to map all writes to a single user.

This avoids ambiguity caused by macOS SMB UID mappings.
