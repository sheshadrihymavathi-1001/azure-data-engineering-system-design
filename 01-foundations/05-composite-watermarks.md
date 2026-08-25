# 05 – Composite Watermarks

## Problem

A timestamp may not uniquely identify a processing position.

Example:

```text
10:00:00 -> Order 101
10:00:00 -> Order 102
10:00:00 -> Order 103
10:00:01 -> Order 104
```

If the pipeline stores only:

```text
updated_at = 10:00:00
```

then a query using:

```sql
WHERE updated_at > '10:00:00'
```

can miss records at exactly `10:00:00`.

## Composite watermark

Use:

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

The timestamp provides the primary ordering and the key provides a deterministic tie-breaker.

## Important distinction

A business key answers:

> Which entity is this?

A watermark answers:

> Up to which source position have I successfully processed?

The same field should not automatically be assumed to serve both purposes.

## Production consideration

Even with a composite watermark, late-arriving or delayed source visibility can still create uncertainty. This is why a small lookback window is often combined with idempotent processing.
