# 11 – Watermark vs CDC vs Snapshot

## Decision matrix

| Capability | Watermark | CDC | Snapshot |
|---|---:|---:|---:|
| Detect inserts | Yes | Yes | Yes |
| Detect updates | Yes* | Yes | Yes |
| Detect deletes | Limited* | Yes | Yes |
| Intermediate changes | No | Yes | No |
| Source change metadata | Required | CDC mechanism | Not required |
| Operational complexity | Low | Medium/High | Medium/High at scale |
| Typical latency | Batch/near-batch | Near-real-time to batch | Batch |
| Large-scale efficiency | Good | Excellent | Potentially expensive |
| Historical change sequence | No | Yes | No |

`*` Depends on source semantics and delete strategy.

## Watermark

Prefer when:

- a reliable modification marker exists
- delete handling is available
- batch/near-batch latency is acceptable
- simplicity matters

Typical design:

```text
updated_at watermark
+
composite tie-breaker
+
lookback
+
idempotent merge
+
delete strategy
```

## CDC

Prefer when:

- reliable change capture is available
- deletes matter
- low latency is important
- change history/replay is required
- source capabilities justify the operational complexity

Typical design:

```text
CDC
+
durable change-position checkpoint
+
ordered processing
+
idempotent target
+
retention/recovery strategy
```

## Snapshot

Prefer when:

- CDC is unavailable
- no reliable modification marker exists
- source cannot be changed
- periodic comparison is acceptable

Typical design:

```text
Yesterday snapshot
+
Today snapshot
+
key comparison
+
row hashes
+
partition/bucket optimization at scale
```

## System-design decision flow

```text
Reliable CDC available?
       |
      YES
       |
      CDC

       NO
       |
Reliable modification marker?
       |
      YES
       |
Watermark
+ composite boundary
+ lookback
+ idempotency
+ delete strategy

       NO
       |
Snapshot comparison
+ hashing
+ partition/bucket optimization
```

## Final principle

Do not choose a mechanism because it is technically fashionable.

Choose based on:

- correctness
- latency
- source capabilities
- data volume
- delete semantics
- recovery requirements
- operational complexity
- cost
- SLA
