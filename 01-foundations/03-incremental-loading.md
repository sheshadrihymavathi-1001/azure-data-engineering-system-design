# 03 – Incremental Loading

## Problem

A full load reads the entire source dataset every time.

For a 500 million-row table:

```text
Run 1 -> 500M rows
Run 2 -> 500M rows
Run 3 -> 500M rows
...
```

If only 1% changes each day, repeatedly moving 500M rows is wasteful.

## Incremental loading

Incremental loading processes only data that changed since the previous successful processing boundary.

Common strategies:

1. Watermark
2. CDC
3. Snapshot comparison
4. Hash comparison

## One-time full load

A full load can still be appropriate for:

- Initial migration
- Initial baseline before CDC
- Small datasets
- Periodic reconciliation

The goal is not "never use full load."

The goal is:

> Avoid unnecessary repeated full loads when a reliable incremental mechanism exists.

## Selection

```text
CDC available?
  -> Prefer CDC when change correctness/latency requires it.

No CDC but reliable updated_at?
  -> Watermark.

Neither available?
  -> Snapshot comparison.

Snapshot comparison:
  -> Use hashes to efficiently detect changed attributes.
```
