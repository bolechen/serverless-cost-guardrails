# Runaway Billing Patterns

Use this catalog to generate hypotheses. Confirm each hypothesis against the target system.

## Recursive work

One job, alarm, webhook, or callback creates another copy of itself.

Signals:

- a queue consumer calls the public enqueue path;
- a user-facing `async` flag reaches the internal worker;
- retry count resets across service boundaries;
- an alarm reschedules before successful completion;
- poison messages return to the same queue forever.

Controls:

- separate enqueue and execute commands;
- carry an immutable job ID and a decreasing attempt count;
- use a dead-letter destination and maximum delivery count;
- reject cyclic workflow transitions;
- cap work per tenant and globally.

## Per-request fan-out

One cheap trigger creates many metered operations. Examples include one row causing many storage writes, one page causing many image transforms, one chat request causing multiple model and search calls, or one webhook broadcasting to every subscriber.

Measure the full multiplication:

`external triggers x retries x items x writes per item x replicas`

Batching reduces unit count but does not replace a total-work limit.

## Fallback becomes the hot path

A fallback performs a full list, table scan, remote fetch, or expensive model call after a cheap index lookup misses. Old records, malformed keys, cache churn, or an incomplete migration can make nearly every request miss.

Require a bounded fallback, a migration completion signal, and a kill switch. Alert on fallback ratio, not only errors.

## Denial of wallet

An attacker spends the owner's money through a legitimate API: AI inference, image transformation, cache fill, exports, email/SMS, authentication, object storage, analytics, or egress.

Authentication alone is insufficient when free accounts are cheap. Apply per-IP, per-principal, per-tenant, and global budgets at the side effect.

## Retry storms and partial failure

Timeouts do not prove the provider did no work. Retrying a timed-out write or send can duplicate the bill and the effect.

Use provider-supported idempotency keys. Persist the claim before the call. Reconcile unknown outcomes instead of immediately replaying them.

Add jitter, exponential backoff, a retry ceiling, and a circuit breaker. Do not retry validation, quota, or permanent authorization failures.

## Bandwidth and hot-object attacks

A small set of large public objects can dominate data transfer. CDN caching does not help when URLs vary, range requests bypass cache, transformations are dynamic, or the CDN itself bills transfer and requests.

Inventory object size, cache key dimensions, range behavior, origin shielding, signed URL policy, and maximum bytes per principal. Rate-limit by bytes when request count hides the real cost.

## Storage accumulation

Unique object names, logs, build artifacts, versions, multipart uploads, and generated exports can grow without a request spike.

Use lifecycle rules, retention classes, per-prefix ownership, incomplete-upload cleanup, and object-count alerts. Verify that deletion code paginates; deleting only the first listing page creates false confidence.

## Query and analytics amplification

Unbounded SQL, public datasets, full-table analytics, wildcard log queries, high-cardinality metrics, and per-event observability can charge by bytes scanned, rows, or events.

Set maximum bytes billed or equivalent limits when supported. Partition and prune before running. Sample noisy telemetry and cap user-controlled date ranges or dimensions.

## Test and automation leakage

Load tests, preview deployments, CI loops, AI coding agents, and synthetic monitors use production-priced resources.

Separate accounts or projects. Give test credentials small quotas. Disable production email, cron, migration, and batch jobs in previews. Put a terminal condition on autonomous agents and workflows.

## Control-plane traps

Stopping the visible service may not stop its storage, snapshots, IPs, logs, replicas, queues, or managed add-ons. A provider budget may only send a notification, or a stop action may exclude some charges.

Test the actual stop procedure before an incident. Document what continues billing and how to restore service safely.
