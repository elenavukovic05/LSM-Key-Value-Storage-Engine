# LSM Key-Value Storage Engine

> An educational key-value storage engine written in Go, built on the LSM-tree architecture used by databases such as Cassandra and RocksDB.

## Overview

This project was developed as a team project (team of 5) as part of the **Advanced Algorithms and Data Structures** course.

The engine stores key-value pairs and supports `PUT`, `GET` and `DELETE`. Writes go to a write-ahead log and an in-memory sorted structure, which is periodically flushed to immutable sorted files on disk. Reads check memory first, then caches, then the files on disk from newest to oldest.

---

# Architecture

```
PUT    ──►  Write-Ahead Log (disk)
                   │
                   ▼
            Memtable (memory, sorted)
                   │  full
                   ▼
              Flush to SSTable
     (data + index + summary + Merkle tree)

GET    ──►  Memtable ──► LRU cache ──► SSTables (newest to oldest)

DELETE ──►  Tombstone record (WAL + memtable)
```

| Component | Role |
|---|---|
| Write-Ahead Log | Every write is persisted first. On startup, unflushed records are replayed into the memtable |
| Memtable manager | Several memtables: one writable, the rest read-only. When all are full, the oldest is flushed as sorted records |
| SSTable | Immutable sorted files with an index and a summary for lookups |
| Merkle tree | Built over SSTable data blocks, used to verify data integrity |
| LRU cache and block cache | Avoid repeated disk reads |
| Token bucket | Rate limiting, its state is stored in the engine itself |

**Why tombstones?** SSTables are immutable, so a deleted key cannot be removed from an existing file. A tombstone record marks it as deleted, and the newest version of a key always wins on reads.

---

# Technologies

- Go
- Write-ahead logging
- B-tree
- Skip list
- Hash map
- Merkle tree
- LRU cache
- Bloom filter
- Token bucket

---

# Memtable Implementations

The memtable type is selected in `config.json` through a common `Memtable` interface:

| Type | Put / Get | Flush |
|---|---|---|
| `hashmap` | O(1) average | needs sorting, O(n log n) |
| `btree` | O(log n) | in-order traversal, O(n) |
| `skiplist` | O(log n) expected | already sorted, O(n) |

---

# Project Features

- `PUT`, `GET` and `DELETE` through a console menu
- Crash recovery by replaying the write-ahead log
- Configurable block size, segment size, memtable size and count, memtable type, cache size and rate limit

---

# My Contribution

This is a fork of a team project. Below are the parts I implemented.

- **Memtable manager** for the hash map, B-tree and skip list variants: rotation of multiple memtables, flushing the oldest one as sorted records, tombstone deletes and WAL recovery
- **Common `Memtable` interface** and config-based selection of the implementation
- **Hash map and B-tree memtables**, and the memtable wrapper around the skip list (the skip list data structure itself was written by a teammate)
- **B-tree**: node search, insertion with sibling rotation and node splitting, root splitting, logical deletion and in-order traversal for sorted export
- **Bloom filter**: implemented with serialization to disk, available on a separate branch and not integrated into the read path

`main.go` was developed jointly by the team.

---

# Known Limitations

- The B-tree order is fixed to 4
- The Bloom filter is not wired into SSTable lookups

---

# Learning Outcomes

This project demonstrates practical experience with:

- Storage engine design and the LSM-tree architecture
- Write-ahead logging and crash recovery
- B-trees, skip lists and hash maps
- Hash-based integrity verification with Merkle trees
- Caching strategies
- Designing components behind a common interface
- Working in a team on a larger codebase

---

# Running

```bash
cd main
go run .
```
