# 07 – Change Data Capture (CDC)

## What is CDC?

CDC captures source changes as changes rather than inferring them from the current table state.

Typical events:

```text
INSERT
UPDATE
DELETE
```

Example:

```text
LSN 100 -> INSERT Order 101
LSN 101 -> UPDATE Order 102
LSN 102 -> DELETE Order 103
LSN 103 -> UPDATE Order 101
```

## Why CDC is powerful

A watermark query asks:

> Which current rows have a modification marker after my watermark?

CDC asks:

> Which changes occurred after my last processed change position?

This makes CDC especially useful for:

- deletes
- high-frequency updates
- low-latency ingestion
- preserving change history
- reliable replay

## Checkpoint

The pipeline persists a source change position, such as an LSN or equivalent source-specific position.

Example:

```text
LSN 100 -> success
LSN 101 -> success
LSN 102 -> success
LSN 103 -> crash
```

Persisted checkpoint:

```text
LSN 102
```

Restart from:

```text
LSN 103
```

## CDC trade-offs

CDC is not automatically perfect. Consider:

- CDC retention
- checkpoint management
- ordering
- transaction boundaries
- schema evolution
- replay/recovery
- operational complexity
- monitoring
- cost

## Important baseline concept

If CDC is enabled after historical data already exists, CDC does not automatically reconstruct all changes that occurred before capture began.

A common migration pattern is:

```text
Initial baseline snapshot
        +
CDC from a known capture position
```

The exact behavior depends on the source technology and retention.
