# Storage Design

This document describes how storage is structured in the system and the reasoning
behind each design choice.

The primary goal of the storage design is to support Time Machine reliably while
minimizing CPU IOWAIT and avoiding unnecessary wear on slower storage devices.

## Design Goals

The storage layout was designed with the following goals in mind:

- Stable Time Machine backups
- Low CPU IOWAIT during backup operations
- Clear separation between active and archival data
- Ability to browse archived data safely
- Cost-effective use of available hardware

## Two-Tier Storage Model

Storage is split into two distinct tiers based on workload characteristics.

### Hot Tier (Active Backups)

The hot tier is used exclusively for active Time Machine backups.

Characteristics:
- DRAM-based SSD
- Low write latency
- Better handling of synchronous writes

Responsibilities:
- Receive all live Time Machine writes
- Handle incremental backups and snapshot rotation
- Absorb sync-heavy and metadata-intensive workloads

This tier is intentionally limited in size and is treated as transient storage.

### Cold Tier (Archive and Files)

The cold tier is used for data that does not require low-latency writes.

Characteristics:
- DRAM-less SSD
- Higher write latency
- Suitable for bulk transfers and infrequent access

Responsibilities:
- Store archived Time Machine sparsebundles
- Store photos and general user files
- Serve data over SMB for browsing

No active Time Machine writes occur on this tier.

## Why Not a Single Disk

Using a single SSD for both active backups and archival storage was tested early
in the setup.

This approach caused the following issues:

- High CPU IOWAIT during backups
- Backup stalls under heavy write pressure
- Degraded system responsiveness

The issue was traced to SSD controller behavior under sync-heavy workloads.
Combining active and cold workloads on the same disk amplified the problem.

## Why RAID Was Avoided

RAID configurations were intentionally not used.

Reasons include:

- RAID does not change controller behavior under synchronous writes
- USB-attached SSDs share bandwidth and controller limitations
- Increased complexity without addressing the root cause
- Higher risk of cascading failures on low-cost hardware

The bottleneck was latency, not throughput.

## Sparsebundle Handling

Each macOS device creates a dedicated sparsebundle on the hot tier.

Important properties of sparsebundles:

- They are treated as atomic units
- They cannot be merged incrementally across destinations
- Partial copying risks corruption

For this reason:
- Sparsebundles are fully replaced during archiving
- No live syncing is performed between tiers

## Archival Strategy

Archiving is performed explicitly and outside the backup window.

Process summary:
- Time Machine writes to the hot tier
- Backups complete and stabilize
- Sparsebundles are moved to the cold tier
- Hot tier space is reclaimed

This ensures:
- Predictable performance during backups
- Minimal wear on the cold tier
- Clear ownership of data at all times

## Capacity Planning

Capacity limits are enforced at the Time Machine level.

Per-device limits:
- Prevent a single device from exhausting hot storage
- Allow Time Machine to prune older snapshots automatically

Cold storage capacity is treated independently and expanded only when required.

## Summary

The storage design favors simplicity and predictability over maximum utilization.

By separating active and archival workloads, the system avoids pathological I/O
patterns and maintains stable performance on modest hardware.
