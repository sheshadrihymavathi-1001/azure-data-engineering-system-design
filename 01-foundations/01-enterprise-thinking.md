# 01 – Enterprise Thinking

## Business problem

A production data platform must move data from operational systems into analytical systems reliably and repeatedly.

The challenge is not simply:

> "How do I copy data?"

The real questions are:

- How much data must be moved?
- How often?
- What happens when the pipeline fails?
- How do we detect inserts, updates, and deletes?
- How do we avoid missing data?
- Can we safely retry?
- Can the solution meet the SLA?
- What will it cost at 10x the current volume?
- How do we secure and monitor it?

## Enterprise design dimensions

Every ingestion design should consider:

| Dimension | Question |
|---|---|
| Correctness | Can records be missed or duplicated? |
| Reliability | What happens after failure? |
| Scalability | What happens when data volume grows 10x? |
| Latency | How quickly must data arrive? |
| Cost | What are the compute, storage, and network costs? |
| Security | Who can access the data and secrets? |
| Operability | How will failures and delays be detected? |
| Recoverability | Can we replay data safely? |

## Core principle

A good architecture is not the architecture with the most technology.

It is the architecture that satisfies the business requirements with acceptable complexity and cost.

## Example

If a small table has a reliable `updated_at` column, a simple watermark may be better than introducing CDC infrastructure.

If a high-volume transactional system requires reliable deletes and near-real-time changes, CDC may justify its additional complexity.

## Interview mindset

Do not start with:

> "I will use Azure Data Factory."

Start with:

> "I need to understand the source, change-detection capability, volume, latency, correctness requirements, and failure semantics."

Then select technology.
