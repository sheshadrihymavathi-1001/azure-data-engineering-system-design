# 09 – Snapshot Comparison

## When to use it

Snapshot comparison is useful when:

- CDC is unavailable
- no reliable modification timestamp exists
- no version/change-tracking column exists
- source systems cannot be changed
- daily or periodic comparison is acceptable

## Basic algorithm

Take:

```text
Yesterday snapshot
Today snapshot
```

Compare keys.

### Inserts

```text
Today keys - Yesterday keys
```

Result:

```text
INSERT
```

### Deletes

```text
Yesterday keys - Today keys
```

Result:

```text
DELETE
```

### Potential updates

Keys present in both snapshots require comparison of their non-key attributes.

Hashing can make this comparison cheaper.

## Important distinction from CDC

Snapshot comparison tells us:

> What appears different between two observations?

CDC tells us:

> What changes actually happened?

Example:

```text
$100 -> $150 -> $100
```

If only the beginning and ending snapshots are captured, snapshot comparison sees no net change.

CDC can preserve the intermediate changes.

## Large-scale challenge

For hundreds of millions of rows, repeated full snapshots can become expensive because of:

- source I/O
- network transfer
- storage
- compute
- comparison time
- daily SLA pressure

## Hierarchical optimization

A scalable comparison can conceptually use:

```text
Partition
   |
Bucket
   |
Row hash
```

If a partition fingerprint is unchanged, skip it.

If it changed, inspect buckets.

If a bucket fingerprint is unchanged, skip it.

If it changed, compare individual rows.

## Partition design matters

Consider:

- distribution
- skew
- stable placement
- partition size
- number of partitions
- metadata overhead
- comparison pattern
