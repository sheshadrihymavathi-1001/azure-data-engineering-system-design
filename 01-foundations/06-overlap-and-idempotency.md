# 06 – Overlap and Idempotency

## Overlap / lookback

A lookback window intentionally re-reads a small amount of previously processed data.

Example:

```text
Last watermark = 01:00
Lookback = 5 minutes

Read from approximately 00:55
```

This protects against:

- late-arriving source changes
- timestamp boundary uncertainty
- delayed visibility
- clock/precision issues

## Why duplicates are acceptable

The overlap intentionally creates duplicate processing.

Therefore:

> Overlap protects correctness; idempotency makes the overlap safe.

## Idempotency

An operation is idempotent when repeating it produces the same final state.

Example:

```text
Process Order 101 once  -> correct
Process Order 101 twice -> same correct state
```

A common target approach is an idempotent upsert/merge using a stable business key.

Conceptually:

```text
Incoming record
      |
      v
Find target key
  /       exists   missing
  |         |
UPDATE     INSERT
```

## Failure recovery

```text
Target commit
      |
      X crash
      |
Checkpoint not advanced
      |
Retry same records
      |
Idempotent merge
```

This gives a practical at-least-once processing model.

## Key principle

> It is often safer to reprocess a small amount of data than to risk permanently missing records.
