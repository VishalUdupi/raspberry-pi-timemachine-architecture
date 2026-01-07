# Lessons Learned

This project surfaced several practical lessons about storage, I/O behavior,
and system design on constrained hardware.

## Key Takeaways

1. SSD health does not imply SSD suitability  
   A disk can be healthy yet perform poorly under sync-heavy workloads.

2. Controller behavior matters more than raw bandwidth  
   USB SSD controllers often become the bottleneck under fsync pressure.

3. Architecture beats tuning  
   Structural changes solved the problem where configuration tweaks did not.

4. Time Machine sparsebundles are atomic  
   They cannot be merged or incrementally archived across destinations.

5. Docker entrypoints are not always the right abstraction  
   Mounting real configuration files can be more reliable than passing flags.

6. Separation of concerns improves stability  
   Isolating backup, archive, and monitoring workloads prevented cascading
   performance issues.

## Final Outcome

By understanding workload characteristics and designing around them, a stable,
low-latency Time Machine system was achieved without requiring high-end hardware
or excessive tuning.
