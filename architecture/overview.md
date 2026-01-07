# Architecture Overview

This setup uses a two-tier storage design to run Time Machine reliably on a Raspberry Pi.

## Storage Layout

The storage is split into two roles.

1. Active backup storage  
   A fast, DRAM-based SSD is used as the live Time Machine destination.  
   All ongoing backups and incremental writes happen here.

2. Archive storage  
   A slower, DRAM-less SSD is used for long-term storage.  
   It holds completed Time Machine sparsebundles and user data such as photos.

Time Machine over SMB generates a large number of sync operations and metadata updates.
Lower-end USB SSD controllers often struggle with this access pattern, which can result in
high CPU IOWAIT and unstable backup performance.  
Separating active backups from archival storage helps avoid this problem.

## Base Platform

- Raspberry Pi 4B  
- Raspberry Pi OS Lite (64-bit)  
- Active SSD: Samsung SSD 860 EVO (250GB, DRAM-based)  
- Archive SSD: EVM SSD (1TB, DRAM-less)

## Software Setup

The system runs on Raspberry Pi OS Lite with services isolated using Docker.

### Containers

1. Time Machine  
   Provides macOS Time Machine backups over SMB.  
   Writes are restricted to the active SSD to keep backup latency low.

2. Archive and file access  
   Exposes archived Time Machine backups as read-only.  
   Provides a writable SMB share for photos and general file storage.

3. Monitoring  
   Runs Glances with the web UI enabled.  
   Used to observe CPU IOWAIT, disk activity, and overall system behavior.
