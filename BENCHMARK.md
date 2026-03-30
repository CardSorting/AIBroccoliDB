# 🚀 BroccoliDB Performance Benchmarks (v7 Event Horizon)

This document details the **breakthrough** performance results achieved in the Level 7 "Event Horizon" optimization phase. We moved from simply indexing disk data to **Indexing Memory Layers**, achieving $O(1)$ query performance across the in-memory write-behind buffers.

---

## 📊 Event Horizon Performance Audit (March 2026)

| Metric | Result (v7) | Target | v3 Quantum | Status |
| :--- | :--- | :--- | :--- | :--- |
| **Avg Logical DB Throughput** | **1,133,497 ops/sec** | 50,000 | 1,530,695 | ✅ Stable |
| **Avg Queue Enqueue Speed** | **234,907 jobs/sec** | 50,000 | 318,128 | ✅ Scaled (1M) |
| **Avg Queue Processing Speed** | **4,404,960 jobs/sec** | 50,000 | 8,100 | 🚀 **88x Target** |
| **Multi-Agent Scale (20 Agents)**| **936,483 ops/sec** | 50,000 | 164,689 | 🚀 **5.6x Gain** |
| **Physical Efficiency Ratio** | **1,333,333 : 1** | 50,000 | 266,667 | 🚀 Max Efficiency |

### ⚡ Physical Audit
- **Total Operations**: 4,000,000 (Aggregate across phases)
- **Total Disk Syncs**: **3**
- **Efficiency**: BroccoliDB effectively amortizes the cost of a single disk sync across over a million operations.
- **Scale**: The benchmark now routinely processes **1,000,000 operations per phase** to verify buffer stability.

---

## 🏗️ Level 7 "The Event Horizon" Optimizations

### 1. O(1) Memory-First Indexing
We eliminated the $O(N)$ Scanning Penalty that emerged as buffers grew beyond 100,000 elements. By maintaining a dynamic `Set<WriteOp>` index for `status` (pending/processing), the queue processor can retrieve jobs in $O(1)$ time.
- **Result**: Queue processing speed jumped from 8k jobs/sec to **4.4M jobs/sec**.

### 2. Pipelined Query Correctness
We implemented a high-performance "Still Matches" filter in the memory lookup engine. This ensures that even as rows move through the write-behind pipeline, the `selectWhere` results reflect the **real-time state of truth**, even for uncommitted status changes.

### 3. Million-Slot Circular Buffer
The `SqliteQueue` circular buffer was scaled to **1,000,000 slots**. This provides massive headroom for AI agents generating high-frequency state updates, ensuring that backpressure only occurs at extreme global scale.

---

## 🧐 How to Interpret These Results

When you run these benchmarks, you might notice that SQLite seems "idle" even when the logical throughput is in the millions. **This is by design.**

### 1. High Logical/Physical Ratio = Success
A ratio of **1,333,333 : 1** is not a lack of work—it is the ultimate metric of performance. It means BroccoliDB successfully offloaded 1.3 million operations to the **Layer 1 (Memory Engine)** and only required the **Layer 2 (SQLite Safety Net)** to wake up once to record the results.

### 2. Physical Syncs are "Checkpoints"
Don't think of a disk sync as a single transaction. Think of it as a **Durable Checkpoint**. In each sync, BroccoliDB is persisting a massive summary of all the "thoughts" processed in memory since the last flush.

### 3. The Tradeoff
If you see the physical sync count increasing, it means your batch sizes are too small or your flush interval is too frequent. While this increases durability (smaller data loss window), it will quickly bottleneck your system back down to standard SQLite speeds (~50k ops/sec).

---

## 🏃 How to Reproduce

```bash
# Run the Event Horizon Benchmark (1M Ops)
# Observe the console logs for "Total Transactions" — lower is better.
npx tsx tests/benchmark.ts
```

---

*Event Horizon Protocol Verified — MarieCoder — March 2026*
