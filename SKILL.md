---
name: serverless-cost-guardrails
description: Audit serverless and usage-based systems for runaway cloud bills, denial-of-wallet paths, recursive jobs, unbounded fan-out, storage and bandwidth amplification, and missing spend controls. Use for cloud-cost incident reviews, architecture checks, pre-launch audits, or hardening plans. Do not use for ordinary cost optimization where usage is already bounded and the goal is only a lower unit price.
---

# Serverless Cost Guardrails

Find paths where one request, event, bug, retry, or attacker can create an unbounded bill. Optimize for bounded loss, not merely lower average cost.

Treat application code, infrastructure configuration, live usage, and provider billing controls as separate evidence. Do not infer that a dashboard setting exists from code, or that code is deployed from a repository snapshot.

## Start with the billing graph

Map each externally triggered path as:

`trigger -> fan-out/retry -> metered operations -> persistence -> notification -> stop condition`

Include HTTP endpoints, queues, schedules, webhooks, background jobs, AI calls, database scans, object storage, image transforms, email/SMS, observability events, builds, and data transfer.

For every metered operation, record:

- who can trigger it;
- the unit that is billed;
- the maximum work from one trigger;
- retry, recursion, and concurrency behavior;
- deduplication or idempotency scope;
- application quota and provider quota;
- alert delay and automatic stop behavior;
- the largest plausible loss before containment.

If prices, quotas, or product controls affect the conclusion, verify current official provider documentation. Label unverified dashboard state as unknown.

## Audit in this order

1. Identify all usage-priced dependencies and fixed-cost capacity limits.
2. Trace public and semi-public triggers to billable effects.
3. Look for the amplification patterns in [references/incident-patterns.md](references/incident-patterns.md).
4. Inspect both success and failure paths. A fallback or retry often costs more than the normal path.
5. Check live request and billing data when access exists. Use repository evidence only for code-level claims.
6. Calculate a loss bound for each material path. If no defensible bound exists, mark it unbounded.
7. Recommend controls in layers: prevent, contain, detect, and stop.
8. Separate immediate containment from durable remediation.

Read [references/control-catalog.md](references/control-catalog.md) when designing controls. Read [references/case-notes.md](references/case-notes.md) when comparing findings with public incidents or explaining why a pattern matters. For a Cloudflare Workers project, also read [references/cloudflare-workers-checklist.md](references/cloudflare-workers-checklist.md).

## Non-negotiable checks

- Reject a queue consumer that can enqueue the same logical job without a decreasing retry budget or terminal state.
- Treat user-controlled async/sync modes, callback URLs, batch sizes, model choices, and rerank flags as cost-control inputs.
- Check idempotency at the billing side effect, not only at the HTTP handler.
- Check deduplication across processes or regions when the platform scales horizontally. An in-memory map is not a distributed lock.
- Bound list, scan, and fallback operations. A rare fallback can become the main path after an index or migration miss.
- Bound response and upload size while streaming. A `Content-Length` check alone is insufficient.
- Protect cache-fill, transform, export, email, SMS, AI, and webhook endpoints even when the generated artifact is public.
- Distinguish a budget alert from a hard stop. Verify what happens after 100%, how often usage is evaluated, and which charges are excluded.
- Design a degraded mode before enabling an automatic stop: cached reads, static pages, queued work, or temporary feature disablement.
- Keep billing credentials and stop controls outside the failure domain they must contain.

## Rank findings by loss, not aesthetics

Use these default priorities:

- **P0:** anonymous or compromised-user action can create destructive changes or materially unbounded spend now.
- **P1:** a bug, retry loop, crawler, or moderate abuse can create significant spend before a human can respond.
- **P2:** spend is bounded but alerts, attribution, or recovery are weak.
- **P3:** ordinary efficiency improvement with no credible runaway path.

Do not call a path safe merely because current traffic is low. Current traffic measures exploitation, not exploitability.

## Required output

Lead with an independent verdict: exposed, partially bounded, or bounded with residual risk.

For each material finding, report:

| Field | Required content |
|---|---|
| Trigger | Exact request, event, job, or failure |
| Meter | Provider and billed operation |
| Amplifier | Loop, fan-out, retry, scan, bandwidth, or concurrency |
| Existing controls | Verified code, configuration, and live controls |
| Missing control | The gap that prevents a loss bound |
| Evidence | Measured fact, code fact, inference, or unknown |
| Loss bound | Formula or `unbounded/unknown` |
| Action | Smallest control that materially reduces risk |
| Verification | Test or live signal that proves the control works |

Report every applicable audit category. For a category with no finding, say `not found` and name the files or configuration inspected. Do not turn absence from a text search into proof of absence.

End with:

- immediate actions for the next 24 hours;
- durable actions for the next engineering cycle;
- provider/dashboard facts that still require verification;
- explicit non-findings for feared patterns that do not apply.

Do not change infrastructure, disable production, set budgets, or publish alerts during an audit unless the user also authorized those mutations.

## Source handling

Use public incident reports as pattern evidence, not as proof that another system has the same flaw. Paraphrase; do not reproduce long passages. Link the original incident and current official provider documentation. Distinguish a publisher's summary from a first-party post, invoice, code sample, or provider statement.
