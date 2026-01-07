# Docker Setup

All services in this system are run using Docker to keep responsibilities isolated
and to make the setup reproducible.

Docker is used for three primary purposes:

- Hosting the Time Machine service
- Exposing archived backups and file storage over SMB
- Monitoring system behavior during backups

Each container has a clearly defined role and does not overlap responsibilities
with other containers.
