# Level 1 – Foundations

This section builds the core reasoning required for enterprise data-engineering system design.

The focus is not on memorizing Azure services. The focus is on understanding how to choose an ingestion strategy based on source capabilities, correctness requirements, scale, latency, cost, and operational constraints.

## Topics

1. [Enterprise Thinking](01-enterprise-thinking.md)
2. [OLTP vs OLAP](02-oltp-vs-olap.md)
3. [Incremental Loading](03-incremental-loading.md)
4. [Watermarks](04-watermarks.md)
5. [Composite Watermarks](05-composite-watermarks.md)
6. [Overlap and Idempotency](06-overlap-and-idempotency.md)
7. [CDC](07-cdc.md)
8. [Checkpointing](08-checkpointing.md)
9. [Snapshots](09-snapshots.md)
10. [Hashing](10-hashing.md)
11. [Watermark vs CDC vs Snapshot](11-watermark-vs-cdc-vs-snapshot.md)

## Core principle

> Choose the simplest ingestion mechanism that satisfies correctness, latency, scale, reliability, and cost requirements.

## Decision flow

```text
Reliable CDC available?
        |
       YES  ---> CDC
        |
       NO
        |
Reliable modification timestamp/version?
        |
       YES  ---> Watermark + overlap + idempotency
        |
       NO
        |
       Snapshot comparison + hashing
```
