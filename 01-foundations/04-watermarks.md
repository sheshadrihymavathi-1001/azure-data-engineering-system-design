# 04 – Watermarks

## Definition

A watermark represents the source boundary up to which data has been successfully processed downstream.

Example:

```text
Last successful watermark = 2026-08-25 01:00:00
```

The next extraction may start after that boundary.

## Critical rule

The watermark must not be advanced simply because extraction succeeded.

Preferred sequence:

```text
Extract
  |
Transform
  |
Validate
  |
Target commit
  |
Watermark/checkpoint commit
```

If the process crashes after target commit but before watermark commit, the records may be processed again.

That is acceptable when target processing is idempotent.

## Failure example

```text
Watermark = 10:00

Read 10:01
Write target
CRASH
Watermark remains 10:00
```

Restart:

```text
Read from 10:00+
```

The 10:01 record may be processed again.

This is safer than advancing the watermark before target commit.

## Requirements for a good watermark

A useful modification marker should be:

- reliably updated when relevant data changes
- sufficiently precise
- consistently populated
- stable enough for incremental extraction
- understood in terms of source semantics

## Important warning

`updated_at` is not automatically reliable.

You must understand what it means in the source system.
