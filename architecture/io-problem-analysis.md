# I/O Problem Analysis

This section documents the I/O issues observed during the initial setup and how the root cause was identified.

## Observed Symptoms

When Time Machine backups were running, the system showed the following behavior:

- CPU IOWAIT frequently spiked between 30% and 60%
- Overall system responsiveness degraded during backups
- Backup speeds were inconsistent and often stalled
- Other services on the Raspberry Pi were affected while backups were active

These symptoms were most noticeable during larger backups or when multiple filesystem operations occurred in parallel.

## Initial Assumptions

The first assumption was that the issue might be related to one of the following:

- Insufficient CPU performance on Raspberry Pi 4B
- Network bandwidth limitations
- Docker overhead
- Samba configuration issues
- Filesystem choice or mount options

However, none of these explanations fully accounted for the observed IOWAIT levels.

## Investigation Process

System monitoring was done using Glances with a web UI enabled.  
The following metrics were closely observed during backups:

- CPU usage vs CPU IOWAIT
- Disk utilization and write latency
- Backup throughput over the network

It became clear that:

- CPU usage was relatively low
- CPU IOWAIT was consistently high
- Disk write operations correlated directly with IOWAIT spikes

This indicated that the CPU was frequently waiting on disk I/O rather than being compute-bound.

## Root Cause Identification

The root cause was traced to the USB-attached SSD used as the Time Machine destination.

Although the SSD itself reported healthy SMART data, its controller exhibited poor behavior under Time Machine workloads. Time Machine over SMB produces:

- Frequent `fsync` calls
- Heavy metadata updates
- Many small, synchronous write operations

Low-cost or DRAM-less USB SSD controllers often struggle with this access pattern. When the controller cannot flush data efficiently, write operations block, causing the kernel to place the CPU into an I/O wait state.

This explained why:

- Disk health appeared normal
- Backup performance was unstable
- CPU IOWAIT was disproportionately high

The problem was not NAND wear or disk failure, but controller limitations under sync-heavy workloads.

## Failed Mitigations

Several approaches were considered or tested but did not resolve the issue:

- Filesystem tuning
- Samba configuration changes
- Docker resource limits
- RAID-style aggregation
- Increasing swap or adjusting memory pressure

These attempts treated symptoms rather than the underlying cause.

## Final Resolution

The issue was resolved by separating workloads based on their I/O characteristics:

- Time Machine writes were redirected to a fast, DRAM-based SSD
- The slower SSD was relegated to cold archival storage only

Once Time Machine backups were moved off the slower SSD:

- CPU IOWAIT dropped to below 3%
- Backup speeds became stable and predictable
- System responsiveness returned to normal

This confirmed that the bottleneck was the storage controller’s handling of synchronous writes, not the platform itself.

## Key Takeaway

High CPU IOWAIT during backups is not necessarily caused by insufficient CPU power or disk health issues.

In this case, the primary factor was SSD controller behavior under sync-heavy workloads.  
Architectural changes, rather than tuning, provided the correct and durable solution.
