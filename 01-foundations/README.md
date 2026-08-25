This README covers practical interview concepts around **incremental loading, watermarks, overlap windows, idempotency, CDC, checkpointing, and large-scale snapshot comparison**.

---
# Level 1 – Foundations

## Incremental Loading

Incremental loading means processing only data that has changed since the previous successful processing boundary instead of repeatedly processing the entire source.

Common strategies:

1. Watermark
2. CDC
3. Snapshot comparison
4. Hash comparison

The appropriate strategy depends on source capabilities and business requirements.

---

## Watermark

A watermark represents the source boundary up to which the target has been successfully processed.

Important principle:

> Never advance the watermark merely because data was read.

The correct sequence is:

Source
→ Extract
→ Transform
→ Validate
→ Target Commit
→ Watermark Commit

The watermark should represent successfully committed target state.

---

## 1. Composite Watermark

A timestamp alone may not uniquely identify a position.

Example:

```text
01:00:00 → Order 101
01:00:00 → Order 102
01:00:00 → Order 103
01:00:01 → Order 104

```sql
updated_at > '01:00:00'
```

could miss **Order 103** if multiple records have the same `updated_at` value and the previous extraction boundary falls within that group.

A composite watermark uses:

```text
(updated_at, primary_key)
```

Conceptually:

```sql
WHERE
    updated_at > :last_updated_at
    OR (
        updated_at = :last_updated_at
        AND order_id > :last_order_id
    )
```

The timestamp provides the primary ordering, while the key provides the tie-breaker.

---

# 2. Lookback / Overlap Window

A small overlap window protects against **late-arriving records and uncertain extraction boundaries**.

Example:

```text
Previous watermark = 01:00

Lookback = 5 minutes

Next extraction:
00:55 → current boundary
```

Some records may be processed more than once.

That is acceptable when downstream processing is **idempotent**.

> It is often safer to process a small amount of data twice than to risk missing data.

---

# 3. Idempotency

An operation is **idempotent** when repeating it produces the same final state.

Example:

```text
Process Order 101 once  → correct state
Process Order 101 twice → same correct state
Process Order 101 ten times → same correct state
```

This allows us to safely use:

```text
At-least-once delivery
        +
Idempotent target processing
```

instead of depending on fragile global exactly-once processing.

Typical target implementation:

```text
Read source changes
       ↓
Apply overlap
       ↓
Deduplicate
       ↓
MERGE / UPSERT
       ↓
Commit target
       ↓
Advance watermark
```

---

# 4. Change Data Capture (CDC)

**Change Data Capture** records source changes rather than inferring changes from the current table state.

CDC can capture:

```text
INSERT
UPDATE
DELETE
```

Example:

```text
LSN 100 → INSERT Order 101
LSN 101 → UPDATE Order 102
LSN 102 → DELETE Order 103
LSN 103 → UPDATE Order 101
```

A reliable CDC pipeline requires:

- Durable checkpointing
- Change ordering
- Idempotent processing
- Retention and recovery strategy
- Monitoring
- Schema evolution handling

---

# 5. Checkpointing

The checkpoint represents the **last source change position known to have been successfully committed downstream**.

Example:

```text
LSN 100 → success
LSN 101 → success
LSN 102 → success
LSN 103 → crash
```

Persisted checkpoint:

```text
LSN 102
```

The pipeline restarts from:

```text
LSN 103
```

If LSN 103 was actually written before the crash but the checkpoint was not updated, LSN 103 will be processed again.

Idempotent processing makes this replay safe.

### Critical ordering

```text
Extract
   ↓
Transform
   ↓
Target Commit
   ↓
Checkpoint / Watermark Commit
```

Never advance the watermark or checkpoint before the target transaction is safely committed.

---

# 6. Snapshot Comparison

When reliable CDC or change-tracking metadata is unavailable, **snapshot comparison** can be used.

Compare:

```text
Yesterday's snapshot
        vs
Today's snapshot
```

### Insert Detection

```text
Today's keys - Yesterday's keys
```

### Delete Detection

```text
Yesterday's keys - Today's keys
```

### Update Detection

Keys existing in both snapshots can be compared using row hashes.

---

# 7. Hash Comparison

Instead of comparing every non-key column individually:

```text
name
address
city
phone
email
salary
status
...
```

Create a row fingerprint:

```text
row_hash = HASH(
    name |
    address |
    city |
    phone |
    email |
    salary |
    status
)
```

Then:

```text
Same key + Same hash
        → No change

Same key + Different hash
        → Update
```

> Hashing simplifies change detection, but it does not eliminate the cost of reading the datasets.

---

# 8. Snapshot vs CDC

Snapshot comparison answers:

> **What appears different between two observations?**

CDC answers:

> **What changes actually happened in the source?**

Consider:

```text
$100 → $150 → $100
```

If both yesterday's and today's snapshots contain:

```text
$100
```

snapshot comparison sees no change.

CDC can preserve both intermediate updates:

```text
UPDATE → $150
UPDATE → $100
```

Therefore, CDC is generally preferable when **change history and accurate delete detection** are required.

---

# 9. Large-Scale Snapshot Comparison

For extremely large datasets, such as **800 million rows**, a hierarchical comparison strategy can reduce unnecessary row-level work.

Conceptually:

```text
Dataset
   ↓
Partition
   ↓
Bucket
   ↓
Row Hash
```

If a partition fingerprint is unchanged:

```text
Skip partition
```

If it changed:

```text
Inspect buckets
```

If a bucket fingerprint is unchanged:

```text
Skip bucket
```

If it changed:

```text
Compare rows
```

This can reduce unnecessary comparison work.

However, partitioning and bucketing must be designed carefully to avoid:

- Data skew
- Extremely large partitions
- Too many tiny partitions
- Excessive metadata
- Unstable data placement

---

# 10. Key Architecture Principles

### Principle 1 — Commit Ordering

> Target commit must happen before watermark/checkpoint advancement.

### Principle 2 — Overlap + Idempotency

> An overlap window is safe when downstream processing is idempotent.

### Principle 3 — Business Key vs Checkpoint

> A business key identifies an entity; a checkpoint identifies a processing position.

### Principle 4 — Timestamp Semantics

> A timestamp is only a reliable watermark when its semantics are suitable for change detection.

### Principle 5 — CDC vs Snapshot

> CDC captures changes; snapshot comparison infers changes.

### Principle 6 — Hashing

> Hashing helps identify changed rows but does not eliminate dataset scanning.

### Principle 7 — At-Least-Once Processing

> At-least-once delivery combined with idempotent processing is often more practical than trying to guarantee global exactly-once processing.

### Principle 8 — Choose Based on Requirements

> Choose the simplest ingestion mechanism that satisfies correctness, latency, scale, reliability, and cost requirements.

---

# Mental Model

```text
                 INCREMENTAL INGESTION
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
   Watermark           CDC           Snapshot
        │                │                │
   Timestamp/        INSERT           Compare
   Composite         UPDATE           snapshots
        │             DELETE               │
        ↓                ↓                ↓
   Lookback         Checkpoint       Hash comparison
        │                │                │
        └────────────────┼────────────────┘
                         ↓
                   Idempotent Target
                         │
                         ↓
                    MERGE / UPSERT
                         │
                         ↓
                Target Commit First
                         │
                         ↓
              Advance Watermark/
                   Checkpoint
```

## Core Interview Takeaway

The key question is not simply:

> **"How do I load only new records?"**

A production-grade design must answer:

```text
How do I detect changes?
How do I avoid missing records?
How do I handle duplicates?
How do I detect deletes?
How do I recover from failures?
How do I replay safely?
How does this scale?
```

That is the difference between a basic incremental pipeline and a **production-grade incremental ingestion architecture**.
