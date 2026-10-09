# Cloudflare Workers Billing Checklist

Use this reference only when the target uses Cloudflare Workers products. Verify enabled bindings and current pricing from `wrangler` configuration, deployed settings, and official documentation. Do not assume that a package dependency means a service is deployed.

For each category, report a finding or `not found`, in addition to the core audit categories in `SKILL.md`. Cite exact files and deployed configuration inspected. Live counts of D1 rows, KV keys, or R2 objects are billed reads; bound them as described in the Boundaries section of `SKILL.md`. Estimate trigger frequency and worst-case monthly billed units before converting units to money.

## Durable Objects and alarms

- Trace every `setAlarm`, alarm callback, retry, and catch/finally path.
- Reject unconditional self-rescheduling. Require a terminal state, maximum attempts, or a next-run decision derived from remaining work.
- Check whether two objects can wake or message each other in a cycle.
- Count storage reads and writes per alarm execution. Include state bookkeeping and acknowledgements.
- Verify that an alarm after partial failure cannot repeat already completed paid work.

## Client polling and scheduled triggers

- Inventory `setInterval`, recursive `setTimeout`, refresh hooks, WebSocket reconnects, and scheduled triggers.
- Require backoff, jitter, an attempt ceiling, request cancellation, and a stop condition.
- Pause browser polling when the page is hidden when freshness permits it.
- Ensure cron work uses a cursor or bounded batch. A schedule must not scan the whole dataset on every run.
- Persist idempotency or progress before an external paid side effect.

## Queues and Worker call graphs

- Draw producer-to-consumer and Worker-to-Worker edges. Reject cycles unless a decreasing budget proves termination.
- Ensure a consumer cannot forward the original asynchronous mode back into the public enqueue path.
- Verify maximum deliveries, dead-letter handling, batch size, partial-batch retry behavior, and poison-message isolation.
- Count operations per message after batching. Batching changes request count, not total downstream work.
- Check service bindings, HTTP callbacks, and tail workers in addition to Queue bindings.

## D1

- Match hot queries to indexes. Use query plans or measured scans rather than naming conventions.
- Flag full scans inside request, queue, alarm, or cron loops.
- Require intentional `WHERE` constraints for `UPDATE` and `DELETE`. Review dynamic query builders for omitted filters.
- Bound result rows, pagination depth, transactions, and retry count.
- Check migrations and legacy rows that can force a fallback scan on most requests.

## KV and Durable Object storage

- Count reads, writes, deletes, and list operations on every hot request path.
- Flag `list()` as a fallback for authentication, lookup, or existence checks.
- Batch storage mutations when semantics permit it.
- Avoid write-on-read patterns for access timestamps, counters, or cache metadata unless sampled or aggregated.
- Bound key cardinality and configure retention for versioned or generated data.

## Workers AI and external AI

- Bound agent steps, tool rounds, refinement loops, retries, output tokens, and parallel model calls.
- Treat client-selected models, rerank flags, reasoning modes, and batch sizes as cost-bearing input.
- Apply per-principal and global token or request budgets before the model call.
- Cache safe deterministic work and use distributed single-flight for expensive cache misses.
- Provide a kill switch and a cache-only or non-AI degraded mode.

## R2, Images, Stream, and bandwidth

- Protect cache-fill, upload, transform, export, and purge endpoints.
- Bound bytes while streaming; do not trust only `Content-Length`.
- Check cache-key cardinality, range requests, variant generation, and whether unique query strings bypass caching.
- Verify lifecycle cleanup paginates through every object.
- Calculate both operation classes and stored/transferred bytes.

## Analytics and logs

- Treat every event, trace, log, and high-cardinality dimension as a possible metered write.
- Sample noisy success paths and cap user-controlled fields.
- Alert on events per user action and writes per request, not only daily totals.

## Verification

Static inspection finds plausible paths. It does not prove termination.

Run these tests only after the user explicitly authorizes them for a named isolated account or environment. Many Cloudflare products have no enforceable per-account spend cap, so use application-level limits and a firm abort condition instead of relying on a provider quota. Never run them against production bindings.

In that environment:

1. make the downstream service fail continuously;
2. deliver a poison queue message;
3. restart the Worker between attempts;
4. overlap two deliveries of the same logical job;
5. advance an alarm through its maximum retry count;
6. observe billed-operation counters and dead-letter state.

The test passes only if work stops at the documented bound and duplicate delivery creates no duplicate paid side effect.
