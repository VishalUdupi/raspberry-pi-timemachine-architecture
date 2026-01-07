# raspberry-pi-timemachine-architecture
Designing a Low-IOWAIT, Tiered Time Machine Backup System on Raspberry Pi


# Low-IOWAIT Time Machine on Raspberry Pi (Tiered Storage Design)

This project documents the design, debugging, and final architecture of a 
low-latency Time Machine backup system built on Raspberry Pi using Docker, 
USB SSDs, and Samba.

## Motivation
- High CPU IOWAIT during Time Machine backups
- Unstable backup speeds
- Cheap USB SSDs failing under sync-heavy workloads

## Goals
- Stable Time Machine backups
- Minimal CPU IOWAIT
- Safe archival storage
- Ability to browse archived data
- Budget-conscious design

## Final Result
- IOWAIT reduced from ~60% → <3%
- Backup throughput ~480 Mbps+
- Clear separation of hot vs cold storage

[Architecture Overview](architecture/overview.md)
[Storage Design](architecture/storage-design.md)
[IOWAIT Problem Analysis](architecture/io-problem-analysis.md)
[Docker Setup](docker/)
[Troubleshooting Guide](troubleshooting/)
[Lessons Learned](lessons-learned.md)
