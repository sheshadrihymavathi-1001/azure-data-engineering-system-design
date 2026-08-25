# 02 – OLTP vs OLAP

## OLTP

Online Transaction Processing systems are optimized for operational transactions.

Examples:

- Orders
- Payments
- Customer updates
- Inventory transactions

Typical characteristics:

- Frequent inserts/updates/deletes
- Low-latency transactions
- Strong transactional consistency
- Normalized schemas
- Many concurrent users

## OLAP

Online Analytical Processing systems are optimized for analysis.

Examples:

- Revenue reporting
- Customer analytics
- Historical trends
- BI dashboards
- Machine-learning datasets

Typical characteristics:

- Large scans
- Aggregations
- Historical data
- Analytical queries
- Columnar/optimized storage

## Why separate them?

An analytical query such as:

```sql
SELECT customer_region, SUM(order_amount)
FROM orders
GROUP BY customer_region;
```

could require scanning a large amount of data.

Running such workloads directly against an OLTP system can compete with business transactions.

A common enterprise pattern is:

```text
OLTP
  |
  | ingestion
  v
Data Platform
  |
  +--> Historical storage
  +--> Curated analytical data
  |
  v
BI / Analytics / ML
```

## System-design takeaway

The analytical platform should reduce the workload and risk placed on the operational system while providing scalable access to historical data.
