---
link: https://devcenter.upsun.com/posts/how-do-we-do-incremental-but-really-full-backups-on-top-of-ceph/
date: 2025-08-25T02:00
excerpt: Learn how Upsun leverages Ceph's RBD export-diff feature to implement high-performance incremental backups that maintain full restore capabilities for large-scale container storage.
slurped: 2025-09-27T11:09
title: "High-performance backups: How Ceph snapshots enable incremental full backups"
tags:
  - ceph
---

## The backup challenge at scale[](#the-backup-challenge-at-scale)

When you’re managing container storage at scale, backup performance becomes critical. Large volumes—those reaching hundreds of gigabytes—can take hours to back up using traditional full-volume streaming methods. This creates operational challenges: extended backup windows, increased resource consumption, and potential data loss exposure during long-running operations.

At Upsun, we face this challenge daily with our Ceph-based storage infrastructure. While Ceph’s copy-on-write (CoW) snapshots provide instant cloning capabilities for our containers, off-site backups require a different approach. We need a solution that combines the speed of incremental backups with the reliability of full restores.

## Why Ceph RBD over traditional file-based approaches[](#why-ceph-rbd-over-traditional-file-based-approaches)

Our storage architecture leverages Ceph’s RADOS Block Device (RBD) feature rather than CephFS for container storage. This choice provides several advantages:

- **Simplified data management**: Working with block devices means handling “bags of bytes” rather than complex file systems
- **Better performance**: Block-level operations eliminate file system overhead
- **Seamless failover**: Containers can migrate across our VM grid without complex file system considerations
- **Snapshot efficiency**: RBD snapshots are instant and space-efficient

While a file-based approach like `rsync` might seem intuitive—comparing file lists and transferring only changed files—it doesn’t align with our block-level storage philosophy.

## The Ceph RBD export-diff solution[](#the-ceph-rbd-export-diff-solution)

Ceph provides an elegant solution through [`rbd export-diff`](https://ceph.io/en/news/blog/2013/incremental-snapshots-with-rbd/), which extracts only the changes between two snapshots at the block level. This feature becomes the foundation for our incremental backup strategy.

Here’s how the basic process works:

1. Create a new RBD snapshot
2. Use `rbd export-diff` to identify changed blocks between snapshots
3. Export only the differential data

However, implementing production-ready backups requires additional considerations beyond the basic diff export.

## Building full restore capability from incremental data[](#building-full-restore-capability-from-incremental-data)

To maintain the ability to perform complete volume restores from blob storage, we developed a chunked metadata system:

### Chunk-based storage architecture[](#chunk-based-storage-architecture)

- **4MB chunks**: Each volume is divided into 4MB blocks for optimal transfer and deduplication
- **Hash-based deduplication**: Chunk keys are generated from content hashes, eliminating duplicate data across the entire system
- **Project-level isolation**: Each project maintains its own chunk catalog to prevent cross-customer data leakage

### Metadata file structure[](#metadata-file-structure)

Each backup generates a metadata file containing:

- Complete list of chunks required for full volume restoration
- Chunk offsets and positions within the volume

This approach ensures that every backup point can restore a complete volume, even though we’re only transferring changed data.

## The backup workflow in practice[](#the-backup-workflow-in-practice)

Here’s the complete backup process:

1. **Snapshot creation**: Generate a new RBD snapshot of the volume
2. **Differential analysis**: Run `rbd export-diff` between the current and previous snapshots
3. **Chunk processing**: Break the differential data into 4MB chunks and generate hashes
4. **Selective upload**: Upload only chunks that don’t already exist in blob storage
5. **Metadata generation**: Create a new metadata file referencing all chunks (new and existing) required for full restoration

This workflow ensures that backup time scales with the amount of changed data rather than total volume size.

## Optimized restore performance[](#optimized-restore-performance)

The hash-based chunk system also accelerates restoration:

1. **Local verification**: Before downloading chunks from blob storage, verify if they already exist locally by comparing hashes
2. **Selective download**: Only retrieve chunks that have changed or are missing locally
3. **Parallel processing**: Multiple chunks can be processed simultaneously

While restore operations still require reading the full volume to verify local chunks, the selective download significantly reduces network transfer time.

## Performance benefits and trade-offs[](#performance-benefits-and-trade-offs)

This architecture delivers substantial improvements:

- **Backup speed**: Scales with data change rate rather than volume size
- **Storage efficiency**: Deduplication reduces storage across all backups
- **Network optimization**: Minimal data transfer for routine backups
- **Restore flexibility**: Any backup point can restore a complete volume

The primary trade-off is complexity—managing chunk metadata and ensuring referential integrity requires more sophisticated backup orchestration than simple volume dumps.

Ceph can enable enterprise-grade backup performance without sacrificing restore capabilities. By leveraging block-level snapshots and implementing intelligent chunking strategies, you can achieve backup speeds that scale with your actual data change patterns.

Ready to experience high-performance, scalable container storage? [Start your free trial](https://upsun.com/) and see how Upsun’s infrastructure handles your most demanding applications.