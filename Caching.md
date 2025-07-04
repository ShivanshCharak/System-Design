# Caching

- The operations which reduces responsetime by saving any heavy computation can come into caching
- Typical Use Cas: Reduces disk I/O or network I/O or compute
- Caches are just glorified  `Hash tables` with some advanced data structure
- Cache are not only RAM based Depends ont he use cases

## Type of Caches
1. CPU Cache (SRAM-Based)

    - Modern CPUs have L1, L2, and L3 caches made of Static RAM (SRAM), which is much faster than DRAM (used for main memory).

    - These are on-chip or near-chip for ultra-low latency.

2. Disk-Based Caching

    - Some systems use SSDs or HDDs as a cache for slower storage (e.g., Linux's bcache, Windows ReadyBoost using USB drives).

    - Database systems (like Redis with disk persistence) may use a mix of RAM and disk for caching.

3. Flash-Based Caching (SSD/NVMe)

    - Enterprise storage systems (e.g., ZFS L2ARC, Oracle Exadata) use SSDs as a secondary cache to speed up frequent data access.

    - Some databases (like MySQL with InnoDB buffer pool extensions) allow SSD caching.

4. Persistent Memory (NVDIMM, Optane)

    - Intel Optane (now discontinued but still in use) provided persistent memory that could act as a cache between RAM and storage.

    - NVDIMMs (Non-Volatile DIMMs) can also serve as a cache.

5. Browser & CDN Caching (Mixed Storage)

    - Web browsers cache files in RAM (for speed) but also on disk (for persistence).

    - CDNs (Content Delivery Networks) may use RAM, SSDs, or even HDDs depending on the access pattern.

6. GPU Cache (VRAM-Based)

    - GPUs have their own cache hierarchy (L1/L2/texture caches) inside VRAM (GDDR/HBM).

7. Distributed Caches (Hybrid Storage)

    - Systems like Apache Ignite or Hazelcast can use a mix of RAM + disk for large-scale caching.

## Why Different Cache Types?

    - Speed vs. Cost vs. Persistence Tradeoff:

        SRAM (fastest, expensive) → DRAM → NVMe/SSD → HDD (slowest, cheapest).

    - Volatility vs. Non-Volatility:

        RAM is fast but loses data on power loss, while SSDs/NVDIMMs retain data.


## What are different Places where we can Cache?

1. Main memory of API server

    - `Limited`:- You cant save huge amount of data there

    
    - `Volatile`: If ApI server crashes whole system crashes


    - `Inconsistency`:- If you are using multiple server each server will have differnt cache data
2. DB Views (materialized)


    - Need to be Reshed again and again if not data will become `STALE`
    - Persistent
    - Best for complex aggregation ( Can be used in dasboards, Read-heavy, write rarely systems)
3. Browser(Local storage)

    - Can stire oersonalized recommendaion in browser
4. CDN
5) DISK of api Server
6) Caching in load balancer/api gateways
