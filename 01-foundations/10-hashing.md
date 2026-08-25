# 10 – Hashing

## Why hashing?

Comparing every column of every common row can be expensive.

For a customer:

```text
customer_id
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

## Comparison

```text
Same key + Same hash
    -> No detected change

Same key + Different hash
    -> Update
```

## What hashing does well

It provides a compact fingerprint for comparing non-key attributes.

It is useful for:

- snapshot comparison
- change detection
- data reconciliation
- hierarchical comparison

## What hashing does NOT do

Hashing does not eliminate the need to read the underlying data.

For 800M rows, you may still need to:

- extract the snapshots
- read the rows
- calculate or read hashes
- compare datasets

Therefore:

> Hashing reduces comparison work; it does not magically eliminate data movement or source scanning.

## Hash design considerations

Be consistent about:

- null handling
- data type normalization
- trimming/formatting
- date/time representation
- delimiter/serialization rules
- column ordering
- excluded technical columns

Otherwise the same logical data can produce different hashes.

## Collision consideration

A hash is a fingerprint, not a mathematical proof of equality.

Choose an appropriate hash function and understand the collision characteristics for the business risk.
