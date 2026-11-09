# Design: fulfillment region on orders, and moving order history to a partitioned table

Status: draft for review. Owner: Orders team. Target: next sprint.

## Background

Warehouse routing reads the shipping address on every order and works out the region at query
time. That lookup is now 30% of the CPU on the routing service. We want to store the region once,
on the order.

Separately, `order_events` (the audit trail of every order state change) has grown to 1.9 billion
rows and 1.4 TB. Vacuum on it runs for most of a day and queries for recent events have slowed. We
want to move it to a table partitioned by month so old months can be detached and archived.

## Current state

- Postgres 15 on a managed instance, one primary and two read replicas.
- `orders`: 420 million rows, about 180 GB, about 900 writes per second at peak, never idle.
- `order_events`: 1.9 billion rows, written by the orders service on every state change.
- The checkout path, the routing service and the support console all read and write `orders`.

## Proposal

### Part A: `fulfillment_region` column

One migration file, run by the deploy pipeline in a single transaction during the Tuesday
afternoon deploy:

```sql
BEGIN;
ALTER TABLE orders ADD COLUMN fulfillment_region text;
UPDATE orders
   SET fulfillment_region = region_for_zip(shipping_zip);
ALTER TABLE orders ALTER COLUMN fulfillment_region SET NOT NULL;
CREATE INDEX orders_fulfillment_region_idx ON orders (fulfillment_region);
COMMIT;
```

The application change that writes `fulfillment_region` on new orders ships in the same deploy.
We expect the migration to take "a few minutes" based on a run against the staging copy, which
has 2 million rows.

### Part B: partitioned `order_events`

1. Create `order_events_v2`, partitioned by month on `created_at`, with the same columns.
2. Change the orders service to write to `order_events_v2` only.
3. Copy history from `order_events` to `order_events_v2` with a background job, one month at a
   time, oldest first.
4. When the copy job reports done, `DROP TABLE order_events` the same night to reclaim the 1.4 TB.
5. Next morning, run a row-count comparison per month between the copy job's log and
   `order_events_v2`, and spot-check 100 orders in the support console.
6. Point the support console and the analytics export at `order_events_v2`.

## Monitoring

- Routing service CPU should drop by about 30% once reads use the stored column.
- We will watch the orders dashboard (error rate, p95 latency) during the deploy.
- The copy job logs rows copied per month.

## Rollback

Part A: if the application change misbehaves, revert the deploy; the column can stay.
Part B: the new write path is a code change and can be reverted by redeploying the previous
build.

## Open items

- Analytics export owners need a heads-up before step 6.
