# Monitoring Container

System monitoring is provided using Glances with its web interface enabled.

## Purpose

- Observe CPU usage and CPU IOWAIT
- Monitor disk utilization during backups
- Validate the effectiveness of architectural changes

## Usage

The container runs with host PID and network access to allow accurate reporting
of system-wide metrics.

Glances was particularly useful in identifying:

- Disk-induced CPU stalls
- The correlation between backup activity and IOWAIT
- Improvements after storage separation
