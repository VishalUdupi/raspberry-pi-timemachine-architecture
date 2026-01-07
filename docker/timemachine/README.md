# Time Machine Container

This container provides macOS Time Machine backups over SMB.

The implementation is based on the `mbentley/timemachine` image, which is designed
specifically for Time Machine compatibility and Apple Time Capsule emulation.

## Purpose

- Act as the primary Time Machine destination for macOS
- Handle live backups and incremental updates
- Write only to fast storage to minimize I/O latency

## Storage Mapping

The container mounts a single host directory:

/tm_stage → /opt/timemachine

This directory resides on a fast, DRAM-based SSD and acts as the active backup
landing zone.

Each macOS device creates its own sparsebundle within this directory.

## UID and GID Mapping

The container explicitly sets UID and GID values:

- TM_UID=1000
- TM_GID=1000

This ensures files written by the container match the host user and prevents
permission issues when accessing backup data outside the container.

## Size Limiting

A per-device size limit is enforced using:

TM_SIZE_LIMIT=150G

This caps the total size of each sparsebundle and allows Time Machine to prune
older snapshots automatically when space is needed.

## Networking

The container runs with host networking enabled.

This avoids additional network translation layers and improves compatibility
with macOS SMB discovery.

## Notes

- The container is intentionally isolated from archival storage
- No manual file access should be performed inside active sparsebundles
- Sparsebundles are treated as atomic units
