# Archive Script

This directory contains scripts used to manage Time Machine archival.

The primary goal of the archive process is to move completed Time Machine
sparsebundles from fast storage to cold storage in a controlled and safe way.

## Why Archiving Is Manual

Time Machine sparsebundles are atomic units and cannot be merged or updated
incrementally once moved to another destination.

Because of this limitation:

- Incremental backups cannot be pushed into an existing archive
- Archiving replaces the entire sparsebundle intentionally
- Partial or live syncing is avoided to prevent corruption

The archive process is therefore explicit and controlled.

## archive_tm.sh

This script performs the following actions:

- Checks whether a Time Machine sparsebundle exists in the staging directory
- Verifies that Time Machine is not actively running
- Replaces any existing archived sparsebundle with the current one
- Frees space on the fast SSD after archiving

The script is designed to fail safely. If any precondition is not met,
no destructive action is taken.

## Execution Model

The script can be run manually or via cron during low-activity hours.

Typical usage is once per day or when staging storage reaches a predefined
threshold.

Archiving is intentionally decoupled from the Time Machine backup process.
