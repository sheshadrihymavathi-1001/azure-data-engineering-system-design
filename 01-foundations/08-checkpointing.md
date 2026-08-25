# 08 – Checkpointing

## Definition

A checkpoint records the last source position known to have been successfully committed downstream.

For CDC:

```text
last_successful_lsn = 102
```

For snapshot processing, the checkpoint may be:

```text
snapshot_id = 20260825
```

## Correct ordering

```text
Read source
   |
Process
   |
Commit target
   |
Commit checkpoint
```

Not:

```text
Commit checkpoint
   |
Commit target
```

## Why?

Bad design:

```text
Checkpoint -> 103
Target processing
CRASH
```

After restart, the pipeline may start at 104 and permanently miss 103.

Safer design:

```text
Target commit
CRASH
Checkpoint remains 102
Restart at 103
Idempotent replay
```

## At-least-once processing

The safe failure model is often:

```text
At-least-once delivery
+
Idempotent target
+
Durable checkpoint
```

This means a record may be processed more than once, but the final target state remains correct.

## Checkpoint requirements

A production checkpoint should be:

- durable
- transactional with respect to the workflow where practical
- auditable
- recoverable
- associated with the source and pipeline version
