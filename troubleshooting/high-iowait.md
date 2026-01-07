# High CPU IOWAIT During Backups

This document describes the symptoms, diagnosis, and resolution of high CPU
IOWAIT observed during Time Machine backups.

## Symptoms

- CPU IOWAIT consistently above 30%
- Backup throughput fluctuating or stalling
- System responsiveness degrading during backups
- Disk activity dominating system metrics

## What IOWAIT Indicates

High IOWAIT means the CPU is frequently blocked waiting for disk operations
to complete. This is not a CPU performance issue, but a storage latency issue.

## Root Cause

The root cause was not disk health or filesystem corruption, but SSD controller
behavior.

Time Machine over SMB generates a workload that includes:

- Frequent synchronous writes
- Metadata-heavy operations
- Repeated fsync calls

Low-cost or DRAM-less USB SSD controllers struggle with this pattern and block
write completion, causing the CPU to wait.

## Why RAID and Tuning Did Not Help

Approaches such as RAID aggregation, filesystem tuning, or Samba configuration
changes did not resolve the issue because they did not address the controller
limitations.

These approaches treated symptoms rather than the underlying cause.

## Resolution

The issue was resolved by redirecting all active Time Machine writes to a fast,
DRAM-based SSD and using the slower SSD only for cold archival storage.

This architectural change reduced CPU IOWAIT to under 3% and stabilized
backup performance.
