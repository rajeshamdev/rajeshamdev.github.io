---
title: 'Erasure Coding: Durability Without the 3x Cost'
description: 'Why erasure coding replaces replication in modern distributed storage, and the tradeoffs that come with it.'
pubDate: 'Apr 13 2026'
---

Replication is simple: store three copies of your data, lose up to two, still be safe. It's also expensive — 200% storage overhead for a durability guarantee most systems can achieve with a fraction of the cost.

Erasure coding is the alternative. Instead of making full copies, you split the data into `k` data chunks and compute `m` parity chunks. Any `k` of the `k+m` chunks can rebuild the original. A typical `(10, 4)` code gives you the ability to lose 4 chunks out of 14 — storage overhead drops to ~40% instead of 200%.

## The catch

Erasure coding is not a free lunch. Three things get harder:

1. **Repair cost.** Reconstructing a lost chunk in a `(10, 4)` code requires reading 10 other chunks — often from 10 different nodes. For a node failure, you multiply that by every chunk the node held. Repair traffic in an EC system is an order of magnitude higher than in a replicated one.
2. **Small-write amplification.** Updating a single byte forces a read-modify-write across the full stripe. Erasure-coded systems almost always pair with log-structured or append-only write paths to avoid this.
3. **Latency tail.** Any read that needs a missing chunk has to decode from parity — which means waiting on the slowest of `k` nodes. This is why most EC systems keep hot data replicated and only EC the cold tier.

## Where I'd use it

...
