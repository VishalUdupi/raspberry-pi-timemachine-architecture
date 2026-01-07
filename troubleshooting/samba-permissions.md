# Samba Permission Issues on macOS

This document covers common permission issues encountered when accessing Samba
shares from macOS.

## Observed Problem

- SMB shares were visible from macOS Finder
- File creation or deletion failed
- Permission errors occurred despite correct credentials

## Root Cause

The issue stemmed from mismatched UID and GID mappings between:

- macOS SMB clients
- Samba running inside Docker
- The Linux host filesystem

macOS does not preserve Linux UID semantics over SMB, which can cause Samba
to write files as unexpected users.

## Failed Approaches

- Running the Samba container as a non-root user
- Passing Samba options through entrypoint flags
- Adjusting permissions without controlling user mapping

These approaches led to inconsistent behavior.

## Final Resolution

The issue was resolved by:

- Running the Samba container as root
- Mounting a custom smb.conf file
- Forcing user and group mapping inside the Samba configuration
- Enforcing ownership and permissions on the host filesystem

Mounting a full smb.conf provided predictable and stable behavior.
